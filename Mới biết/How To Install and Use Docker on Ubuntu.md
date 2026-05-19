## [Step 1 — Installing Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-1-installing-docker)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-1-installing-docker)

The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. To do that, we’ll add a new package source, add the GPG key from Docker to ensure the downloads are valid, and then install the package.

First, update your existing list of packages:

```
sudo apt update
```

Next, install a few prerequisite packages which let `apt` use packages over HTTPS:

```
sudo apt install ca-certificates curl gnupg
```

Then add the GPG key for the official Docker repository to your system using the modern keyring method:

```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Add the Docker repository to APT sources. This command automatically detects your Ubuntu version:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update the package database with the Docker packages from the newly added repo:

```
sudo apt update
```

Verify that you are about to install from the Docker repository instead of the default Ubuntu repository:

```
apt-cache policy docker-ce
```

You’ll see output indicating that Docker is available from the Docker repository. The version number will vary, but you should see the Docker repository URL in the output.

Finally, install Docker:

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Note:** The installation includes `docker-buildx-plugin` and `docker-compose-plugin` which provide modern Docker Compose functionality. You can use `docker compose` (without the hyphen) instead of the older standalone `docker-compose` command.

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:

```
sudo systemctl status docker
```

The output should be similar to the following, showing that the service is active and running:

```
Output● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-05-19 17:00:41 UTC; 17s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 24321 (dockerd)
      Tasks: 8
     Memory: 46.4M
     CGroup: /system.slice/docker.service
             └─24321 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

Installing Docker now gives you not just the Docker service (daemon) but also the `docker` command line utility, or the Docker client. We’ll explore how to use the `docker` command later in this tutorial.

## [Step 2 — Executing the Docker Command Without Sudo (Optional)](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-2-executing-the-docker-command-without-sudo-optional)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-2-executing-the-docker-command-without-sudo-optional)

By default, the `docker` command can only be run the **root** user or by a user in the **docker** group, which is automatically created during Docker’s installation process. If you attempt to run the `docker` command without prefixing it with `sudo` or without being in the **docker** group, you’ll get an output like this:

```
Outputdocker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

If you want to avoid typing `sudo` whenever you run the `docker` command, add your username to the `docker` group:

```
sudo usermod -aG docker ${USER}
```

To apply the new group membership, log out of the server and back in, or type the following:

```
su - ${USER}
```

You will be prompted to enter your user’s password to continue.

Confirm that your user is now added to the **docker** group by typing:

```
groups
```

```
Outputsammy sudo docker
```

If you need to add a user to the `docker` group that you’re not logged in as, declare that username explicitly using:

```
sudo usermod -aG docker username
```

The rest of this article assumes you are running the `docker` command as a user in the **docker** group. If you choose not to, please prepend the commands with `sudo`.

Let’s explore the `docker` command next.

## [Step 3 — Using the Docker Command](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-3-using-the-docker-command)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-3-using-the-docker-command)

Using `docker` consists of passing it a chain of options and commands followed by arguments. The syntax takes this form:

```
docker [option] [command] [arguments]
```

To view all available subcommands, type:

```
docker
```

As of Docker 19, the complete list of available subcommands includes:

```
Output  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

```

To view the options available to a specific command, type:

```
docker docker-subcommand --help
```

To view system-wide information about Docker, use:

```
docker info
```

Let’s explore some of these commands. We’ll start by working with images.

## [Step 4 — Working with Docker Images](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-4-working-with-docker-images)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-4-working-with-docker-images)

Docker containers are built from Docker images. By default, Docker pulls these images from [Docker Hub](https://hub.docker.com/), a Docker registry managed by Docker, the company behind the Docker project. Anyone can host their Docker images on Docker Hub, so most applications and Linux distributions you’ll need will have images hosted there.

To check whether you can access and download images from Docker Hub, type:

```
docker run hello-world
```

The output will indicate that Docker in working correctly:

```
OutputUnable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

Docker was initially unable to find the `hello-world` image locally, so it downloaded the image from Docker Hub, which is the default repository. Once the image downloaded, Docker created a container from the image and the application within the container executed, displaying the message.

You can search for images available on Docker Hub by using the `docker` command with the `search` subcommand. For example, to search for the Ubuntu image, type:

```
docker search ubuntu
```

The script will crawl Docker Hub and return a listing of all images whose name match the search string. In this case, the output will be similar to this:

```
OutputNAME                                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                                                    Ubuntu is a Debian-based Linux operating sys…   10908               [OK]
dorowu/ubuntu-desktop-lxde-vnc                            Docker image to provide HTML5 VNC interface …   428                                     [OK]
rastasheep/ubuntu-sshd                                    Dockerized SSH service, built on top of offi…   244                                     [OK]
consol/ubuntu-xfce-vnc                                    Ubuntu container with "headless" VNC session…   218                                     [OK]
ubuntu-upstart                                            Upstart is an event-based replacement for th…   108                 [OK]
ansible/ubuntu14.04-ansible                               Ubuntu 14.04 LTS with
...

```

In the **OFFICIAL** column, **OK** indicates an image built and supported by the company behind the project. Once you’ve identified the image that you would like to use, you can download it to your computer using the `pull` subcommand.

Execute the following command to download the official `ubuntu` image to your computer:

```
docker pull ubuntu
```

You’ll see the following output:

```
OutputUsing default tag: latest
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete
fc878cd0a91c: Pull complete
6154df8ff988: Pull complete
fee5db0ff82f: Pull complete
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

After an image has been downloaded, you can then run a container using the downloaded image with the `run` subcommand. As you saw with the `hello-world` example, if an image has not been downloaded when `docker` is executed with the `run` subcommand, the Docker client will first download the image, then run a container using it.

To see the images that have been downloaded to your computer, type:

```
docker images
```

The output will look similar to the following:

```
OutputREPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              1d622ef86b13        3 weeks ago         73.9MB
hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
```

As you’ll see later in this tutorial, images that you use to run containers can be modified and used to generate new images, which may then be uploaded (_pushed_ is the technical term) to Docker Hub or other Docker registries.

Let’s look at how to run containers in more detail.

## [Step 5 — Running a Docker Container](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-5-running-a-docker-container)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-5-running-a-docker-container)

The `hello-world` container you ran in the previous step is an example of a container that runs and exits after emitting a test message. Containers can be much more useful than that, and they can be interactive. After all, they are similar to virtual machines, only more resource-friendly.

As an example, let’s run a container using the latest image of Ubuntu. The combination of the **-i** and **-t** switches gives you interactive shell access into the container:

```
docker run -it ubuntu
```

Your command prompt should change to reflect the fact that you’re now working inside the container and should take this form:

```
Outputroot@d9b100f2f636:/#
```

Note the container id in the command prompt. In this example, it is `d9b100f2f636`. You’ll need that container ID later to identify the container when you want to remove it.

Now you can run any command inside the container. For example, let’s update the package database inside the container. You don’t need to prefix any command with `sudo`, because you’re operating inside the container as the **root** user:

```
apt update
```

Then install any application in it. Let’s install Node.js:

```
apt install nodejs
```

This installs Node.js in the container from the official Ubuntu repository. When the installation finishes, verify that Node.js is installed:

```
node -v
```

You’ll see the version number displayed in your terminal:

```
Outputv10.19.0
```

Any changes you make inside the container only apply to that container.

To exit the container, type `exit` at the prompt.

Let’s look at managing the containers on our system next.

## [Step 6 — Managing Docker Containers](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-6-managing-docker-containers)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-6-managing-docker-containers)

After using Docker for a while, you’ll have many active (running) and inactive containers on your computer. To view the **active ones**, use:

```
docker ps
```

You will see output similar to the following:

```
OutputCONTAINER ID        IMAGE               COMMAND             CREATED

```

In this tutorial, you started two containers; one from the `hello-world` image and another from the `ubuntu` image. Both containers are no longer running, but they still exist on your system.

To view all containers — active and inactive, run `docker ps` with the `-a` switch:

```
docker ps -a
```

You’ll see output similar to this:

```
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 8 seconds ago                       quizzical_mcnulty
a707221a5f6c        hello-world         "/hello"            6 minutes ago       Exited (0) 6 minutes ago                       youthful_curie

```

To view the latest container you created, pass it the `-l` switch:

```
docker ps -l
```

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 40 seconds ago                       quizzical_mcnulty
```

To start a stopped container, use `docker start`, followed by the container ID or the container’s name. Let’s start the Ubuntu-based container with the ID of `1c08a7a0d0e4`:

```
docker start 1c08a7a0d0e4
```

The container will start, and you can use `docker ps` to see its status:

```
OutputCONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         3 minutes ago       Up 5 seconds                            quizzical_mcnulty

```

To stop a running container, use `docker stop`, followed by the container ID or name. This time, we’ll use the name that Docker assigned the container, which is `quizzical_mcnulty`:

```
docker stop quizzical_mcnulty
```

Once you’ve decided you no longer need a container anymore, remove it with the `docker rm` command, again using either the container ID or the name. Use the `docker ps -a` command to find the container ID or name for the container associated with the `hello-world` image and remove it.

```
docker rm youthful_curie
```

You can start a new container and give it a name using the `--name` switch. You can also use the `--rm` switch to create a container that removes itself when it’s stopped. See the `docker run help` command for more information on these options and others.

Containers can be turned into images which you can use to build new containers. Let’s look at how that works.

## [Step 7 — Committing Changes in a Container to a Docker Image](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-7-committing-changes-in-a-container-to-a-docker-image)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-7-committing-changes-in-a-container-to-a-docker-image)

When you start up a Docker image, you can create, modify, and delete files just like you can with a virtual machine. The changes that you make will only apply to that container. You can start and stop it, but once you destroy it with the `docker rm` command, the changes will be lost for good.

This section shows you how to save the state of a container as a new Docker image.

After installing Node.js inside the Ubuntu container, you now have a container running off an image, but the container is different from the image you used to create it. But you might want to reuse this Node.js container as the basis for new images later.

Then commit the changes to a new Docker image instance using the following command.

```
docker commit -m "What you did to the image" -a "Author Name" container_id repository/new_image_name
```

The **-m** switch is for the commit message that helps you and others know what changes you made, while **-a** is used to specify the author. The `container_id` is the one you noted earlier in the tutorial when you started the interactive Docker session. Unless you created additional repositories on Docker Hub, the `repository` is usually your Docker Hub username.

For example, for the user **sammy**, with the container ID of `d9b100f2f636`, the command would be:

```
docker commit -m "added Node.js" -a "sammy" d9b100f2f636 sammy/ubuntu-nodejs
```

When you _commit_ an image, the new image is saved locally on your computer. Later in this tutorial, you’ll learn how to push an image to a Docker registry like Docker Hub so others can access it.

Listing the Docker images again will show the new image, as well as the old one that it was derived from:

```
docker images
```

You’ll see output like this:

```
OutputREPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
sammy/ubuntu-nodejs   latest              7c1f35226ca6        7 seconds ago       179MB
...

```

In this example, `ubuntu-nodejs` is the new image, which was derived from the existing `ubuntu` image from Docker Hub. The size difference reflects the changes that were made. And in this example, the change was that NodeJS was installed. So next time you need to run a container using Ubuntu with NodeJS pre-installed, you can just use the new image.

You can also build Images from a `Dockerfile`, which lets you automate the installation of software in a new image. However, that’s outside the scope of this tutorial.

Now let’s share the new image with others so they can create containers from it.

## [Step 8 — Pushing Docker Images to a Docker Repository](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-8-pushing-docker-images-to-a-docker-repository)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-8-pushing-docker-images-to-a-docker-repository)

The next logical step after creating a new image from an existing image is to share it with a select few of your friends, the whole world on Docker Hub, or other Docker registry that you have access to. To push an image to Docker Hub or any other Docker registry, you must have an account there.

This section shows you how to push a Docker image to Docker Hub. To learn how to create your own private Docker registry, check out [How To Set Up a Private Docker Registry on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-18-04) or [How To Set Up a Private Docker Registry on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-20-04).

To push your image, first log into Docker Hub.

```
docker login -u docker-registry-username
```

You’ll be prompted to authenticate using your Docker Hub password. If you specified the correct password, authentication should succeed.

**Note:** If your Docker registry username is different from the local username you used to create the image, you will have to tag your image with your registry username. For the example given in the last step, you would type:

```
docker tag sammy/ubuntu-nodejs docker-registry-username/ubuntu-nodejs
```

Then you may push your own image using:

```
docker push docker-registry-username/docker-image-name
```

To push the `ubuntu-nodejs` image to the **sammy** repository, the command would be:

```
docker push sammy/ubuntu-nodejs
```

The process may take some time to complete as it uploads the images, but when completed, the output will look like this:

```
OutputThe push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Pushed
5f70bf18a086: Pushed
a3b5c80a4eba: Pushed
7f18b442972b: Pushed
3ce512daaf78: Pushed
7aae4540b42d: Pushed
...
```

After pushing an image to a registry, it should be listed on your account’s dashboard, like that show in the image below.

![New Docker image listing on Docker Hub](https://assets.digitalocean.com/articles/docker_1804/ec2vX3Z.png)

If a push attempt results in an error of this sort, then you likely did not log in:

```
OutputThe push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Preparing
5f70bf18a086: Preparing
a3b5c80a4eba: Preparing
7f18b442972b: Preparing
3ce512daaf78: Preparing
7aae4540b42d: Waiting
unauthorized: authentication required
```

Log in with `docker login` and repeat the push attempt. Then verify that it exists on your Docker Hub repository page.

You can now use `docker pull ==sammy==/==ubuntu-nodejs==` to pull the image to a new machine and use it to run a new container.

## [Docker vs Docker Compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#docker-vs-docker-compose)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#docker-vs-docker-compose)

While Docker lets you build and run containers, managing multi-container applications with just Docker can be tedious. That’s where **Docker Compose** comes in.

**Docker Compose** is a tool for defining and running multi-container Docker applications using a YAML file. Instead of manually running separate containers for a web server, database, and caching layer, you can define them all in a `docker-compose.yml` file and run them with:

```
docker-compose up
```

For a detailed guide on using Docker Compose, including how to set up complex multi-container applications and manage their configurations, check out our tutorial on [How To Install and Use Docker Compose on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04). This guide will walk you through creating and managing multi-container applications with Docker Compose, from basic setups to more complex configurations.

**Note:** Modern Docker installations include Docker Compose as a plugin. Use `docker compose` (without the hyphen) instead of the standalone `docker-compose` command. The plugin provides the same functionality with better integration.

|Feature|Docker CLI|Docker Compose|
|---|---|---|
|Usage|Single container operations|Multi-container orchestration|
|Configuration|CLI commands|YAML configuration file|
|Dependency handling|Manual|Handles linked services automatically|
|Best use case|Testing isolated containers|Local development and staging setups|

For more, see [How To Install and Use Docker Compose on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04).

## [Troubleshooting Common Docker Installation Issues](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#troubleshooting-common-docker-installation-issues)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#troubleshooting-common-docker-installation-issues)

### [Problem: `docker: command not found`](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#problem-docker-command-not-found)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#problem-docker-command-not-found)

**Fix:** The Docker CLI isn’t in your `$PATH`. Reinstall Docker or ensure `/usr/bin` is included.

```
sudo apt install docker-ce docker-ce-cli containerd.io
```

### [Problem: `Cannot connect to the Docker daemon`](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#problem-cannot-connect-to-the-docker-daemon)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#problem-cannot-connect-to-the-docker-daemon)

**Fix:** Docker isn’t running, or your user isn’t in the `docker` group.

```
sudo systemctl start docker
sudo usermod -aG docker $USER
```

Then log out and log back in.

### [Problem: GPG key or repository error](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#problem-gpg-key-or-repository-error)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#problem-gpg-key-or-repository-error)

**Fix:** If you encounter GPG key or repository errors, refer to [Docker’s official installation documentation](https://docs.docker.com/engine/install/ubuntu/) for the latest key and repository steps. The modern installation method using `/etc/apt/keyrings` should work across Ubuntu 20.04, 22.04, and 24.04. For automated installation that can help avoid these issues, consider using our guide on [How To Use Ansible to Install and Set Up Docker on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-install-and-set-up-docker-on-ubuntu-20-04).

## [Docker Desktop on Ubuntu (Beta)](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#docker-desktop-on-ubuntu-beta)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#docker-desktop-on-ubuntu-beta)

Docker Desktop is now available in beta for Linux distributions like Ubuntu. It provides a GUI, bundled Docker Engine, and Kubernetes support.

To install Docker Desktop on Ubuntu:

```
sudo apt install ./docker-desktop-<version>-<arch>.deb
```

Refer to [Docker Desktop for Linux docs](https://docs.docker.com/desktop/install/linux/) for prerequisites and download links.

> **Note:** Docker Desktop is suited for development environments. For server-side installs, use Docker CE.

## [Installing Docker Using a Dockerfile](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#installing-docker-using-a-dockerfile)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#installing-docker-using-a-dockerfile)

For automated environments, install Docker using a `Dockerfile`. Here’s an example:

```
FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y ca-certificates curl gnupg && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && \
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

For more advanced automation options, you can also use Ansible to install and configure Docker. Check out our guide on [How To Use Ansible to Install and Set Up Docker on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-install-and-set-up-docker-on-ubuntu-20-04). This is particularly useful if you need to manage Docker installations across multiple servers or want to maintain consistent configurations in your infrastructure.

## [How to Uninstall Docker on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#how-to-uninstall-docker-on-ubuntu)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#how-to-uninstall-docker-on-ubuntu)

To remove Docker from your system, run:

```
sudo apt purge docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

## [Frequently Asked Questions](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#frequently-asked-questions)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#frequently-asked-questions)

### [1. What is the best way to install Docker on Ubuntu?](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#1-what-is-the-best-way-to-install-docker-on-ubuntu)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#1-what-is-the-best-way-to-install-docker-on-ubuntu)

The recommended and most reliable way to install Docker on Ubuntu is by using Docker’s official repository. This approach ensures you always receive the latest stable version, complete with security patches and updates directly from Docker. The installation process uses the modern GPG keyring method (`/etc/apt/keyrings`) which is compatible with Ubuntu 20.04, 22.04, and 24.04. After adding the repository and updating your package index, you can install Docker Engine and related components using:

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

This method is preferred over using the default Ubuntu repositories, which may contain outdated versions. By following Docker’s official installation steps, you ensure compatibility with the latest features, bug fixes, and security enhancements, making your Docker experience on Ubuntu both robust and secure.

### [2. Do I need sudo to run Docker commands?](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#2-do-i-need-sudo-to-run-docker-commands)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#2-do-i-need-sudo-to-run-docker-commands)

By default, Docker requires root privileges to run its commands, which means you need to prepend `sudo` to every Docker command. However, you can avoid this by adding your user to the `docker` group, which grants permission to run Docker commands without `sudo`. To do this, execute

```
sudo usermod -aG docker $USER
```

and then log out and log back in for the changes to take effect. This step modifies your user’s group memberships, allowing Docker to be run as a non-root user. Keep in mind that adding users to the `docker` group grants them root-level access to the Docker daemon, so only add trusted users. If you skip this step, you’ll encounter permission errors when running Docker commands without `sudo`. For security and convenience, it’s best to add your user to the `docker` group if you use Docker frequently.

### [3. How do I verify if Docker is running correctly?](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#3-how-do-i-verify-if-docker-is-running-correctly)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#3-how-do-i-verify-if-docker-is-running-correctly)

To confirm that Docker is installed and running properly on your Ubuntu system, you can use a couple of simple commands. First, run

```
docker info
```

to display detailed information about the Docker installation, including server version, storage driver, and running containers. If Docker is running, this command will return a wealth of information; if not, you’ll see an error. Another common test is to run

```
docker run hello-world
```

This command downloads a test image from Docker Hub and runs it in a container. If everything is set up correctly, you’ll see a message indicating that Docker is working as expected. If you encounter errors, check that the Docker service is running with

```
sudo systemctl status docker
```

and review any error messages for troubleshooting. These steps help ensure your Docker installation is functional and ready for use.

### [4. Can I install a specific version of Docker?](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#4-can-i-install-a-specific-version-of-docker)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#4-can-i-install-a-specific-version-of-docker)

Yes, you can install a specific version of Docker on Ubuntu, which is useful if you need compatibility with certain applications or want to avoid the latest changes. First, list all available Docker versions in the repository using

```
apt-cache madison docker-ce
```

This command will display a list of version strings. Once you identify the desired version, install it by running

```
sudo apt install docker-ce=<VERSION_STRING>
```

replacing `<VERSION_STRING>` with the exact version number (for example, `5:20.10.7~3-0~ubuntu-focal`). This approach ensures you have precise control over the Docker version on your system. Remember to also specify matching versions for

```
docker-ce-cli
```

and

```
containerd.io
```

if needed. Pinning a specific version can help maintain stability in production environments or when working with legacy projects that require a particular Docker release.

### [5. What’s the difference between Docker and Docker Compose?](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#5-what-s-the-difference-between-docker-and-docker-compose)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#5-what-s-the-difference-between-docker-and-docker-compose)

Docker and Docker Compose are related but serve different purposes. Docker is a platform that allows you to build, ship, and run individual containers, each encapsulating an application and its dependencies. It’s ideal for running single services or applications in isolation. Docker Compose, on the other hand, is a tool designed to define and manage multi-container applications. With Compose, you use a YAML file (`docker-compose.yml`) to specify how multiple containers should interact, their configurations, networks, and volumes. This makes it easy to orchestrate complex applications consisting of several interconnected services, such as a web server, database, and cache. While Docker alone is sufficient for simple use cases, Docker Compose streamlines the process of managing multi-container setups, making development, testing, and deployment of complex applications much more efficient and reproducible.

**Note:** Modern Docker installations include Docker Compose as a plugin. Use `docker compose` (without the hyphen) instead of the standalone `docker-compose` command. For more information, see [How To Install and Use Docker Compose on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04).

### [6. How do I remove Docker from my system?](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#6-how-do-i-remove-docker-from-my-system)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#6-how-do-i-remove-docker-from-my-system)

To completely uninstall Docker from your Ubuntu system, you need to remove the Docker packages and delete any associated data. Start by running

```
sudo apt purge docker-ce docker-ce-cli containerd.io
```

to remove the Docker Engine, CLI, and container runtime. This command deletes the installed packages but leaves configuration files and data directories intact. To fully clean up, remove Docker’s data directories with

```
sudo rm -rf /var/lib/docker
```

and

```
sudo rm -rf /var/lib/containerd
```

These directories store images, containers, volumes, and other persistent data. If you added your user to the `docker` group, you may also want to remove the group with

```
sudo groupdel docker
```

After these steps, Docker and all its data will be removed from your system. Always back up important data before uninstalling to avoid accidental loss.

### [7. Is Docker Desktop available for Ubuntu, and should I use it?](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#7-is-docker-desktop-available-for-ubuntu-and-should-i-use-it)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#7-is-docker-desktop-available-for-ubuntu-and-should-i-use-it)

Yes, Docker Desktop is now available in beta for Linux distributions, including Ubuntu. Docker Desktop provides a graphical user interface (GUI) for managing containers, images, volumes, and networks, making it easier for users who prefer not to work exclusively with the command line. It also bundles the Docker Engine and offers integrated Kubernetes support, which is useful for development and testing environments. To install Docker Desktop on Ubuntu, download the appropriate `.deb` package from Docker’s official website and install it using

```
sudo apt install ./docker-desktop-<version>-<arch>.deb
```

However, Docker Desktop is primarily intended for development and not recommended for production servers. For server-side or headless environments, it’s better to use Docker Engine Community Edition (CE) via the command line. Always check Docker’s documentation for prerequisites and the latest installation instructions before proceeding.

## [Conclusion](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#conclusion)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#conclusion)

In this comprehensive tutorial, you’ve successfully installed Docker on Ubuntu, mastered the fundamentals of container management, and learned how to work with Docker images and containers. You’ve also gained hands-on experience with Docker Hub by pushing a modified image to the registry. These skills form a solid foundation for container-based development and deployment.

The installation steps provided work across Ubuntu 20.04, 22.04, and 24.04, using the modern GPG keyring method for secure package verification. By following Docker’s official repository setup, you ensure you receive the latest stable releases with security updates.

### [Next steps](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#next-steps)[](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#next-steps)

To continue learning and building with Docker, explore these related resources:

- **[How To Install and Use Docker Compose on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04)**: Learn how to manage multi-container applications with Docker Compose.
    
- **[The Docker Ecosystem: An Introduction to Common Components](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components)**: Deepen your understanding of Docker’s architecture and components.
    
- **[How To Use Ansible to Install and Set Up Docker on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-install-and-set-up-docker-on-ubuntu-20-04)**: Automate Docker installation across multiple servers using Ansible.
    
- **[DigitalOcean App Platform](https://www.digitalocean.com/products/app-platform)**: Deploy containerized applications without managing infrastructure.
    
- **[DigitalOcean Kubernetes](https://www.digitalocean.com/products/kubernetes)**: Scale your containerized applications with managed Kubernetes orchestration.
    
- Explore more [Docker tutorials](https://www.digitalocean.com/community/tags/docker?type=tutorials) in the DigitalOcean Community covering advanced topics like container orchestration, networking, and security best practices.
    

When you’re ready to deploy, consider using [DigitalOcean Droplets](https://www.digitalocean.com/products/droplets) to run your Docker containers on high-performance cloud infrastructure.

Thanks for learning with the DigitalOcean Community. Check out our offerings for compute, storage, networking, and managed databases.

[Learn more about our products](https://www.digitalocean.com/products "Learn more about our products")

### About the author

![Anish Singh Walia](https://www.gravatar.com/avatar/c91039d94d0ab6e82a10aaf08e3c1c341d5744c8533ff6d077ee4a765246d8d3?default=retro&size=256)

Anish Singh Walia

Author

Sr Technical Content Strategist and Team Lead

[See author profile](https://www.digitalocean.com/community/users/asinghwalia)

I help Businesses scale with AI x SEO x (authentic) Content that revives traffic and keeps leads flowing | 3,000,000+ Average monthly readers on Medium | Sr Technical Writer(Team Lead) @ DigitalOcean | Ex-Cloud Consultant @ AMEX | Ex-Site Reliability Engineer(DevOps)@Nutanix

Category:

[Tutorial](https://www.digitalocean.com/community/tutorials?subtype=tutorial)

Tags:

[Ubuntu](https://www.digitalocean.com/community/tags/ubuntu)

[DigitalOcean App Platform](https://www.digitalocean.com/community/tags/digitalocean-app-platform)

[Docker](https://www.digitalocean.com/community/tags/docker)

[](https://twitter.com/intent/tweet?url=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Fhow-to-install-and-use-docker-on-ubuntu-20-04%3Futm_medium%3Dcommunity%26utm_source%3Dtwshare%26utm_content%3Dhow-to-install-and-use-docker-on-ubuntu-20-04&text=&via=digitalocean "Share to X (Twitter)")[](https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Fhow-to-install-and-use-docker-on-ubuntu-20-04%3Futm_medium%3Dcommunity%26utm_source%3Dtwshare%26utm_content%3Dhow-to-install-and-use-docker-on-ubuntu-20-04&t= "Share to Facebook")[](https://www.linkedin.com/shareArticle?mini=true&url=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Fhow-to-install-and-use-docker-on-ubuntu-20-04%3Futm_medium%3Dcommunity%26utm_source%3Dtwshare%26utm_content%3Dhow-to-install-and-use-docker-on-ubuntu-20-04&title= "Share to LinkedIn")[](https://news.ycombinator.com/submitlink?u=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Fhow-to-install-and-use-docker-on-ubuntu-20-04%3Futm_medium%3Dcommunity%26utm_source%3Dtwshare%26utm_content%3Dhow-to-install-and-use-docker-on-ubuntu-20-04&t= "Share to YCombinator")

#### Still looking for an answer?

[Ask a question](https://www.digitalocean.com/community/questions)[Search for more help](https://www.digitalocean.com/community)

Was this helpful?

YesNo

Comments(10)Follow-up questions(0)

[](https://www.digitalocean.com/community/markdown "Help")

﻿

This textbox defaults to using Markdown to format your answer.

You can type !ref in this text area to quickly search our full set of tutorials, documentation & marketplace offerings and insert the link!

[Sign in/up to comment](https://www.digitalocean.com/api/dynamic-content/v1/login?success_redirect=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Fhow-to-install-and-use-docker-on-ubuntu-20-04%23step-2-executing-the-docker-command-without-sudo-optional&error_redirect=https%3A%2F%2Fwww.digitalocean.com%2Fauth-error&type=register)

![stefanRun](https://www.gravatar.com/avatar/363b3e16c268fcd3c0b93d7b538f7713b4a8d837ba3e7f403216ae93fe237f9c?default=retro)

[stefanRun](https://www.digitalocean.com/community/users/stefanrun)

[June 11, 2020](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=88699)

Show less

I have installed Ubuntu 20.04 server and am struggling to install Docker. I am getting stuck on the step to add the GPG key for the official Docker repository; when I run this command:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

I get this response:

```
curl: (7) Failed to connect to download.docker.com port 443: No route to host
gpg: no valid OpenPGP data found.
```

I can’t work out how to move forward, so any ideas would be appreciated, thanks in advance.

Reply

(1) replies

![Hugh R](https://www.gravatar.com/avatar/27c81231ec88c4e1a03273c9d6ff33b5414296a1888985f1586258b3e7bf7764?default=retro)

[Hugh R](https://www.digitalocean.com/community/users/digitaloceanc0564cab85270d)

[June 23, 2020](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=89095)

Show less

I have a brand new droplet and there appears to be something wrong with the “stable” release using these instructions. When I run `sudo systemctl status docker` I get a bunch of error and it throws me into Vi:

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-06-23 20:47:38 AEST; 15s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 20339 (dockerd)
      Tasks: 8
     Memory: 34.7M
     CGroup: /system.slice/docker.service
             └─20339 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.156277373+10:00" level=warning msg="Your kernel does not support cgroup rt runtime"
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.156436943+10:00" level=warning msg="Your kernel does not support cgroup blkio weigh>
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.156588335+10:00" level=warning msg="Your kernel does not support cgroup blkio weigh>
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.156960819+10:00" level=info msg="Loading containers: start."
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.420014263+10:00" level=info msg="Default bridge (docker0) is assigned with an IP ad>
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.643617453+10:00" level=info msg="Loading containers: done."
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.729797979+10:00" level=info msg="Docker daemon" commit=48a66213fe graphdriver(s)=ov>
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.730768864+10:00" level=info msg="Daemon has completed initialization"
Jun 23 20:47:38 projects systemd[1]: Started Docker Application Container Engine.
Jun 23 20:47:38 projects dockerd[20339]: time="2020-06-23T20:47:38.776191749+10:00" level=info msg="API listen on /run/docker.sock"
```

Although the last message claims Docker has started, I can’t run any containers. Any suggestions would be great.

Reply

![tjonezs54s](https://www.gravatar.com/avatar/cd3e91062f0016c614ad7cc1246d58f50b2c921165b5007c9a51370dd9bc501e?default=retro)

[tjonezs54s](https://www.digitalocean.com/community/users/tjonezs54s)

[January 15, 2021](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=95160)

Show less

Thx… Nice Demo… worked right out of the box… easy to follow…

Reply

![attand](https://www.gravatar.com/avatar/79860dea7001446676a141514b86d928359f68a6913341d3d0429815f78bfdef?default=retro)

[attand](https://www.digitalocean.com/community/users/attand)

[January 18, 2021](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=95210)

Show less

sudo add-apt-repository “deb [arch=amd64] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) focal stable”

When I run the above command on an Ubuntu 20.4 instance on AWS, then i get the following response

"E: The repository ‘[https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) focal Release’ does not have a Release file. N: Updating from such a repository can’t be done securely, and is therefore disabled by default. N: See apt-secure(8) manpage for repository creation and user configuration details. " I am trying this on January 2021, about 6 months after the article. Yet the Release file is not available.

How should we proceed

Reply

![khashashin](https://www.gravatar.com/avatar/ef22ebc96562311de3c5ae20cd98922a23536e04922cc2cc727936016b777828?default=retro)

[khashashin](https://www.digitalocean.com/community/users/khashashin)

[February 27, 2021](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=96368)

Show less

Thx very helpful tutorial. However I got error during apt-key command

```
Warning: apt-key is deprecated.
```

Reply

![swapansanjay](https://www.gravatar.com/avatar/14022245cbc0cbc3bee9d2a155028c0093172c7f57bb1a64c8cc917405abf4c5?default=retro)

[swapansanjay](https://www.digitalocean.com/community/users/swapansanjay)

[April 15, 2021](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=97697)

Show less

I have installed ubuntu 20.04 from ubuntu official website and Docker desktop from Docker offical website. When I try checking the status of docker using the command - sudo systemctl status docker It fails with the message - System has not been booted with systemd as init system (PID 1). Can’t operate. Failed to connect to bus: Host is down I would request Brian Hogan(original poster of this page) to help solve this issue first as rest of the process does not hold any meaning for me. This issue of systemctl and systemd not available with WSL2 is a general issue and needs to be fixed first before venturing into installing docker through ubuntu 20.04 Thanks.

Reply

(1) replies

![Manuel Avelar](https://www.gravatar.com/avatar/da200e5bcbf67847bb87f29a3dca4e348998c64a360aa1ead9030b15c5f9f806?default=retro)

[Manuel Avelar](https://www.digitalocean.com/community/users/pixelcreart)

[July 16, 2021](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=99829)

Show less

Anyone knows why this error?

```
E: gnupg, gnupg2 and gnupg1 do not seem to be installed, but one of them is required for this operation
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   983  100   983    0     0   8776      0 --:--:-- --:--:-- --:--:--  8776
(23) Failed writing body
```

Reply

![hadesunseenn](https://www.gravatar.com/avatar/cc052b1341060b1963bba5c11cda8fc64eec485b832432a71145d00519a47b0f?default=retro)

[hadesunseenn](https://www.digitalocean.com/community/users/hadesunseenn)

[November 21, 2021](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=102615)

Show less

Thanks for sharing, worked great.

Reply

![adsk2050](https://www.gravatar.com/avatar/e8d04c5d6d13e86c1a6771be55d70fd37117198c3b009969723172d598340ed5?default=retro)

[adsk2050](https://www.digitalocean.com/community/users/adsk2050)

[March 3, 2022](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=170805)

This comment has been deleted

![abdulaziz bin saeed](https://www.gravatar.com/avatar/5f41d3cce5937cf48031d9b228b165c826390d9fb20215e06d1cefbb8d959645?default=retro)

[abdulaziz bin saeed](https://www.digitalocean.com/community/users/azizsa03)

[April 27, 2022](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04?comment=172514)

Show less

After running the Command:

> docker run hello-world

I Get this massage:

> docker: error pulling image configuration: download failed after attempts=6: net/http: TLS handshake timeout.

“Ubuntu 20.04.4 LTS”

Reply

Load more comments

[![Creative Commons](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fcreativecommons.c0a877f1.png&width=384)](https://creativecommons.org/licenses/by-nc-sa/4.0/)This work is licensed under a Creative Commons Attribution-NonCommercial- ShareAlike 4.0 International License.

## Deploy on DigitalOcean

Click below to sign up for DigitalOcean's virtual machines, Databases, and AIML products.

[Sign up](https://cloud.digitalocean.com/registrations/new?refcode=f6fcd01aaffb)

## Popular Topics

1. [AI/ML](https://www.digitalocean.com/community/tags/ai-ml)
2. [Ubuntu](https://www.digitalocean.com/community/tags/ubuntu)
3. [Linux Basics](https://www.digitalocean.com/community/tags/linux-basics)
4. [JavaScript](https://www.digitalocean.com/community/tags/javascript)
5. [Python](https://www.digitalocean.com/community/tags/python)
6. [MySQL](https://www.digitalocean.com/community/tags/mysql)
7. [Docker](https://www.digitalocean.com/community/tags/docker)
8. [Kubernetes](https://www.digitalocean.com/community/tags/kubernetes)
9. [All tutorials](https://www.digitalocean.com/community/tutorials)
10. [Talk to an expert](https://www.digitalocean.com/company/contact/sales?referrer=tutorials)

### Connect on Discord

Join the conversation in our Discord to connect with fellow developers

[Visit Discord](https://discord.gg/digitalocean)

## Featured tutorials

1. [SOLID Design Principles Explained: Building Better Software Architecture](https://www.digitalocean.com/community/tutorials/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
2. [How To Remove Docker Images, Containers, and Volumes](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes)
3. [How to Create a MySQL User and Grant Privileges (Step-by-Step)](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)

- [All tutorials](https://www.digitalocean.com/community/tutorials)
- [All topic tags](https://www.digitalocean.com/community/tags)

![](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Ftutorials-2-tulip.764b9f59.svg&width=1920)

## Become a contributor for community

Get paid to write technical tutorials and select a tech-focused charity to receive a matching donation.

[Sign Up](https://www.digitalocean.com/community/pages/write-for-digitalocean)

![](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fdocs-2-kiwi.239a03ef.svg&width=1920)

## DigitalOcean Documentation

Full documentation for every DigitalOcean product.

[Learn more](https://docs.digitalocean.com/)

![](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fblogs-1-lavender.495d1f00.svg&width=1920)

## Resources for startups and AI-native businesses

The Wave has everything you need to know about building a business, from raising funding to marketing your product.

[Learn more](https://www.digitalocean.com/resources)

## Get our newsletter

Stay up to date by signing up for DigitalOcean’s Infrastructure as a Newsletter.

Submit

New accounts only. By submitting your email you agree to our [Privacy Policy](https://www.digitalocean.com/legal/privacy-policy)

## The developer cloud

Scale up as you grow — whether you're running one virtual machine or ten thousand.

[View all products](https://www.digitalocean.com/products)

![](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fclouds-mobile.5d14bead.svg&width=3840)

## Start building today

From GPU-powered inference and Kubernetes to managed databases and storage, get everything you need to build, scale, and deploy intelligent applications.

[Sign up](https://cloud.digitalocean.com/registrations/new)

![](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fwaves-mobile.a054c63e.svg&width=3840)

## Company

- [About](https://www.digitalocean.com/about)
- [Leadership](https://www.digitalocean.com/leadership/executive-management)
- [Blog](https://www.digitalocean.com/blog)
- [Careers](https://www.digitalocean.com/careers)
- [Customers](https://www.digitalocean.com/customers)
- [Partners](https://www.digitalocean.com/partners)
- [Referral Program](https://www.digitalocean.com/referral-program)
- [Affiliate Program](https://www.digitalocean.com/affiliates)
- [Press](https://www.digitalocean.com/press)
- [Legal](https://www.digitalocean.com/legal)
- [Privacy Policy](https://www.digitalocean.com/legal/privacy-policy)
- [Security](https://www.digitalocean.com/security)
- [Investor Relations](https://investors.digitalocean.com/)

## Products

- [GPU Droplets](https://www.digitalocean.com/products/gradient/gpu-droplets)
- [Bare Metal GPUs](https://www.digitalocean.com/products/gradient/bare-metal-gpus)
- [Inference Engine](https://www.digitalocean.com/products/inference-engine)
- [Data & Learning](https://www.digitalocean.com/data-learning)
- [Droplets](https://www.digitalocean.com/products/droplets)
- [Kubernetes](https://www.digitalocean.com/products/kubernetes)
- [Functions](https://www.digitalocean.com/products/functions)
- [App Platform](https://www.digitalocean.com/products/app-platform)
- [Load Balancers](https://www.digitalocean.com/products/load-balancers)
- [Managed Databases](https://www.digitalocean.com/products/managed-databases)
- [Spaces](https://www.digitalocean.com/products/spaces)
- [Block Storage](https://www.digitalocean.com/products/block-storage)
- [Network File Storage](https://www.digitalocean.com/products/storage/network-file-storage)
- [API](https://docs.digitalocean.com/reference/api)
- [Uptime](https://www.digitalocean.com/products/uptime-monitoring)
- [Cloud Security Posture Management (CSPM)](https://www.digitalocean.com/products/cloud-security-posture-management)
- [Identity and Access Management (IAM)](https://www.digitalocean.com/products/identity-access-management)
- [Cloudways](https://www.digitalocean.com/products/cloudways)
- [View all Products](https://www.digitalocean.com/products)

## Resources

- [Community Tutorials](https://www.digitalocean.com/community/tutorials)
- [Community Q&A](https://www.digitalocean.com/community/questions)
- [CSS-Tricks](https://css-tricks.com/)
- [Write for DOnations](https://www.digitalocean.com/community/pages/write-for-digitalocean)
- [Currents Research](https://www.digitalocean.com/currents)
- [DigitalOcean Startups](https://www.digitalocean.com/startups)
- [Wavemakers Program](https://www.digitalocean.com/wavemakers)
- [Compass Council](https://www.digitalocean.com/research)
- [Open Source](https://www.digitalocean.com/open-source)
- [Newsletter Signup](https://www.digitalocean.com/community#iaan)
- [Marketplace](https://www.digitalocean.com/products/marketplace)
- [Pricing](https://www.digitalocean.com/pricing)
- [Pricing Calculator](https://www.digitalocean.com/pricing/calculator)
- [Documentation](https://docs.digitalocean.com/)
- [Release Notes](https://docs.digitalocean.com/release-notes)
- [Code of Conduct](https://www.digitalocean.com/community/pages/code-of-conduct)
- [Shop Swag](https://store.digitalocean.com/)

## Solutions

- [AI GPU Hosting](https://www.digitalocean.com/solutions/ai-gpu-hosting)
- [H100 Cloud GPU](https://www.digitalocean.com/solutions/h100-cloud-gpu)
- [AI Training GPU](https://www.digitalocean.com/solutions/ai-training-gpu)
- [GPU Inference](https://www.digitalocean.com/solutions/gpu-inference)
- [VPS Hosting](https://www.digitalocean.com/solutions/vps-hosting)
- [Website Hosting](https://www.digitalocean.com/solutions/website-hosting)
- [VPN](https://www.digitalocean.com/solutions/vpn)
- [Docker Hosting](https://www.digitalocean.com/solutions/docker-hosting)
- [Node.js Hosting](https://www.digitalocean.com/solutions/nodejs-hosting)
- [Web Mobile Apps](https://www.digitalocean.com/solutions/web-mobile-apps)
- [WordPress Hosting](https://www.digitalocean.com/solutions/wordpress-hosting)
- [Virtual Machines](https://www.digitalocean.com/solutions/virtual-machines)
- [View all Solutions](https://www.digitalocean.com/solutions)

## Contact

- [Support](https://www.digitalocean.com/support)
- [Sales](https://www.digitalocean.com/company/contact/sales?referrer=footer)
- [Report Abuse](https://www.digitalocean.com/company/contact/abuse)
- [System Status](https://status.digitalocean.com/)
- [Share your ideas](https://ideas.digitalocean.com/)

© 2026 DigitalOcean, LLC.[Sitemap](https://www.digitalocean.com/sitemap).Cookie Preferences

- [](https://twitter.com/digitalocean "X (Twitter)")
- [](https://www.instagram.com/thedigitalocean/ "Instagram")
- [](https://www.facebook.com/DigitalOceanCloudHosting "Facebook")
- [](https://discord.gg/digitalocean "Discord")
- [](https://www.youtube.com/DigitalOcean "YouTube")
- [](https://www.linkedin.com/company/digitalocean/ "LinkedIn")
- [](https://github.com/digitalocean "GitHub")
- [](https://www.glassdoor.com/Overview/Working-at-DigitalOcean-EI_IE823482.11,23.htm "Glassdoor")
- [](https://www.builtinnyc.com/company/digitalocean "BuiltInNYC")

How To Install and Use Docker on Ubuntu | DigitalOcean