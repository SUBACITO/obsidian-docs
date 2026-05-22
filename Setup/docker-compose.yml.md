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
    command: --interval 30 --label-enable
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

networks:
  shared-network:
    external: true

# Các service muốn updated thì gắn label vào`
services:
  blog-test:
    image: hehe12312123/blog-test:latest
    container_name: blog-test
    restart: unless-stopped
    ports:
      - "3000:3000"
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
    networks:
      - shared-network

networks:
  shared-network:
    external: true
