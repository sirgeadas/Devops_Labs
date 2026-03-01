# VProfile Project – Containers Intro

> 📁 Part 3 of my DevOps learning path — previous: [Automated Deployment](../02-vprofile-automation/README.md)

## Overview

The goal here is simple: take the same VProfile app stack I previously deployed manually across **five separate VMs**, and run the whole thing on **a single VM using containers**. With one command!

## Architecture

In the manual and automated deployments, each service ran on its own VM:

| VM       | Service    | Role                    |
| -------- | ---------- | ----------------------- |
| `web01`  | Nginx      | Reverse proxy           |
| `app01`  | Tomcat     | Java application server |
| `db01`   | MySQL      | Relational database     |
| `mc01`   | Memcached  | Caching layer           |
| `rmq01`  | RabbitMQ   | Message broker          |

In this project, all five services run as **containers on a single VM**, orchestrated by Docker Compose.

```
Host VM
└── Docker Engine
    ├── nginx       (container) → port 80
    ├── tomcat      (container) → port 8080
    ├── mysql       (container) → port 3306
    ├── memcached   (container) → port 11211
    └── rabbitmq    (container) → port 5672
```
## Prerequisites

- [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)
- Git Bash (or equivalent terminal)
- At least **2 GB of free RAM**

> The Vagrant file for this section installs Docker Engine and Docker CLI automatically.


## Step 1: VM Setup

- Create a folder and copy the appropriate Vagrant file from the course resources:
  - `Vagrantfile` — Windows / macOS Intel

- Check and clean up any running VMs first:
```bash
vagrant global-status --prune
```

- If any VMs are running, halt them:
```bash
# Navigate to the VM's folder first
vagrant halt
```

- Bring up the container VM:
```bash
vagrant up
```

> ⚠️ This will take a few minutes on first run. It installs Docker Engine, Docker CLI, and related packages automatically.

- SSH into the VM and switch to root:
```bash
vagrant ssh
sudo -i
```

## Step 2: Get the Docker Compose File

The `docker-compose.yml` file is available in this project folder.

- Create a working directory and download the file:
```bash
mkdir compose && cd compose
wget https://raw.githubusercontent.com/sirgeadas/Devops_Labs/refs/heads/main/03-vprofile-container/docker-compose.yml
```

The compose file defines all five services and references pre-built images hosted on Docker Hub under the `vprocontainers` account. 

In the future, I'll build these images from scratch.


## Step 3: Run the Application

- Start all containers in detached mode:
```bash
docker compose up -d
```

Docker will pull the required images on first run and start all five containers. This may take a few minutes.

- Verify all containers are running:
```bash
docker compose ps
```

- Verify the images that were pulled:
```bash
docker images
```

## Validation

- Get the VM's IP address:
```bash
ip addr show
```

Use any `192.168.x.x` address from the output.

- Open a browser and navigate to:
```
http://192.168.x.x
```

You should see the VProfile login page served by the Nginx container.

- Log in with:
  - **Username:** `admin_vp`
  - **Password:** `admin_vp`

Run through the following checks to validate each container:

| Action | Validates |
| ------ | --------- |
| Successful login | MySQL container |
| RabbitMQ tab loads | RabbitMQ container |
| User list loads from cache | Memcached container |
| Pages served correctly | Nginx + Tomcat containers |


## Cleanup

- Stop and remove all containers:
```bash
docker compose down
```

- Remove all unused images and containers (full cleanup):
```bash
docker system prune -a
```

- Exit and power off the VM:
```bash
exit
vagrant halt
```

> The compose file and VM remain intact. Run `docker compose up -d` again anytime to bring the stack back up.

---

## Takeaways

The manual deployment required me to:
- Configure five VMs
- Install and secure each service individually
- Manage firewall rules
- Set the startup order right

 The automated deployment scripted all of that, but still provisioned full VMs.

But here, the same application stack came up with a single command.

That gap — five VMs and hours of work vs. one command — is why the industry shifted to containers. Can't wait for the next chapters! 😊

