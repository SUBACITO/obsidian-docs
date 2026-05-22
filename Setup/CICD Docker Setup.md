# CI/CD Docker Setup

#docker #cicd #github-actions #watchtower

## Tổng quan Flow

```
Developer push PR
    → GitHub Actions build Docker image
    → Push lên DockerHub
    → Watchtower detect image mới
    → Tự động restart container
```

---

## 1. GitHub Actions Workflows

### main.yml — Trigger khi merge vào main

```yaml
name: BlogTest-Main

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        include:
          - image_name: blog-test
            dockerfile: Dockerfile
            context: .
    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4

      - name: Validate Dockerfile exists
        run: |
          if [ ! -f "${{ matrix.dockerfile }}" ]; then
            echo "Missing Dockerfile: ${{ matrix.dockerfile }}"
            exit 1
          fi

      - name: Login to Registry
        uses: docker/login-action@v4
        with:
          registry: ${{ secrets.DOCKERHUB_REGISTRY }}
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push ${{ matrix.image_name }}
        uses: docker/build-push-action@v7
        with:
          context: ${{ matrix.context }}
          file: ${{ matrix.dockerfile }}
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_REGISTRY }}/${{ secrets.DOCKERHUB_USERNAME }}/${{ matrix.image_name }}:latest
            ${{ secrets.DOCKERHUB_REGISTRY }}/${{ secrets.DOCKERHUB_USERNAME }}/${{ matrix.image_name }}:sha-${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### pr.yml — Trigger khi tạo Pull Request vào main

```yaml
name: BlogTest-PR

on:
  pull_request:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        include:
          - image_name: blog-test
            dockerfile: Dockerfile
            context: .
    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4

      - name: Validate Dockerfile exists
        run: |
          if [ ! -f "${{ matrix.dockerfile }}" ]; then
            echo "Missing Dockerfile: ${{ matrix.dockerfile }}"
            exit 1
          fi

      - name: Login to Registry
        uses: docker/login-action@v4
        with:
          registry: ${{ secrets.DOCKERHUB_REGISTRY }}
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push ${{ matrix.image_name }}
        uses: docker/build-push-action@v7
        with:
          context: ${{ matrix.context }}
          file: ${{ matrix.dockerfile }}
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_REGISTRY }}/${{ secrets.DOCKERHUB_USERNAME }}/${{ matrix.image_name }}:test
            ${{ secrets.DOCKERHUB_REGISTRY }}/${{ secrets.DOCKERHUB_USERNAME }}/${{ matrix.image_name }}:pr-${{ github.event.pull_request.number }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### So sánh 2 workflow

||`main.yml`|`pr.yml`|
|---|---|---|
|Trigger|push vào `main`|tạo PR vào `main`|
|Tags|`:latest`, `:sha-xxx`|`:test`, `:pr-5`|
|Mục đích|Production|Test/Review|

---

## 2. GitHub Secrets

Vào **Repo → Settings → Secrets and variables → Actions** → thêm 3 secrets:

|Secret|Giá trị|
|---|---|
|`DOCKERHUB_REGISTRY`|`docker.io`|
|`DOCKERHUB_USERNAME`|username DockerHub|
|`DOCKERHUB_TOKEN`|Personal Access Token DockerHub|

### Tạo DockerHub Token

1. Vào **hub.docker.com → Account Settings → Personal access tokens**
2. Generate new token → quyền **Read, Write, Delete**
3. Copy token → paste vào `DOCKERHUB_TOKEN`

---

## 3. GitHub Organization & Collaborators

### Add contributor vào repo

**Cách 1 — Direct access:**

> Repo → Settings → Collaborators and teams → Add people → chọn **Write**

**Cách 2 — Qua Team (khuyên dùng):**

> Organization → Teams → New team → Add members → Add repositories → chọn **Write**

> [!NOTE] Add vào Organization thôi chưa đủ. Phải grant quyền trực tiếp vào repo mới push được.

---

## 4. Docker Compose Stack

### Cấu trúc thư mục

```
/home/pipilabu/system/
├── auth/
│   └── htpasswd          # credentials cho registry local
├── data/                 # registry data
├── config.json           # Docker auth credentials
└── docker-compose.yml
```

### docker-compose.yml

```yaml
services:
  registry:
    image: registry:2
    container_name: registry
    restart: always
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    volumes:
      - ./auth:/auth
      - ./data:/var/lib/registry
    networks:
      - shared-network

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    environment:
      DOCKER_API_VERSION: "1.40"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./config.json:/config.json
    command: blog-test --interval 30
    networks:
      - shared-network

  dozzle:
    image: amir20/dozzle:latest
    container_name: dozzle
    restart: unless-stopped
    ports:
      - "5580:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - shared-network

  blog-test:
    image: hehe12312123/blog-test:latest
    container_name: blog-test
    restart: unless-stopped
    ports:
      - "3000:3000"
    networks:
      - shared-network

networks:
  shared-network:
    external: true
```

---

## 5. Setup Registry Local với Auth

### Tạo htpasswd

```bash
sudo apt install apache2-utils -y
mkdir auth
htpasswd -Bc auth/htpasswd <username>
```

### Tạo config.json

```bash
# Encode credentials
echo -n "username:password" | base64
```

```json
{
  "auths": {
    "localhost:5000": {
      "auth": "base64_local_registry"
    },
    "https://index.docker.io/v1/": {
      "auth": "base64_dockerhub"
    }
  }
}
```

> [!WARNING] `config.json` phải là **file**, không phải thư mục. Dùng `touch config.json` hoặc `nano config.json`, không dùng `mkdir config.json`!

---

## 6. Watchtower

Tự động pull image mới và restart container khi có update.

```bash
# Xem log watchtower
docker logs watchtower -f
```

Log bình thường:

```
Watchtower 1.7.1
Only checking containers which name matches "blog-test"
Scheduling first run: 2026-05-20 08:15:50
```

> [!TIP] `command: blog-test --interval 30` → chỉ watch container tên `blog-test`, check mỗi 30 giây. Bỏ `blog-test` trong command nếu muốn watch tất cả container.

---

## 7. Các lỗi hay gặp

|Lỗi|Nguyên nhân|Fix|
|---|---|---|
|`push access denied`|Sai credentials hoặc token thiếu quyền Write|Tạo lại token với quyền Read, Write, Delete|
|`not found` tag latest|Chưa có tag `:latest` trên DockerHub|Dùng tag `:test` hoặc thêm `:latest` vào workflow|
|`client version 1.25 is too old`|Watchtower cần API version cao hơn|Thêm `DOCKER_API_VERSION: "1.40"`|
|`pull access denied`|Repo private, chưa login|`docker login -u <username>`|
|Watchtower không restart|Thiếu `command: <container_name>`|Thêm `command: blog-test --interval 30`|