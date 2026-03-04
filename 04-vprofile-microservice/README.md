# EMart App – Microservices with Docker Compose

> 📁 Part 4 of my DevOps learning path — previous: [Containers Intro](../03-vprofile-containers/README.md)


## Overview

After running the VProfile monolithic app on containers, this project takes it a step further with a **microservices architecture**. 
EMart: A simplified e-commerce app, where each piece of the stack is its own independent service.

What's the difference from Part 3? 
Here, Docker Compose doesn't just pull pre-built images, it **builds them from source** before running the containers. 
One command still! But now...it compiles the whole app too!


## Architecture

EMart uses an API Gateway pattern. 
Nginx sits at the front and routes traffic to the right service based on the URL path:

| Endpoint | Service | Language | Database |
| -------- | ------- | -------- | -------- |
| `/` (root) | Client App | Angular | — |
| `/api` | Mart API | Node.js | MongoDB |
| `/webapi` | Books API | Java | MySQL |

```

Browser

└── Nginx (API Gateway) → port 80

    ├── /          → Angular Client App (container)

    ├── /api       → Node.js Mart API (container)

    │                   └── MongoDB (container)

    └── /webapi    → Java Books API (container)

                        └── MySQL (container)

```

All six services run as containers on a single VM, orchestrated by Docker Compose.


## Prerequisites

- [Oracle VM VirtualBox](https://www.virtualbox.org/)

- [Vagrant](https://www.vagrantup.com/)

- Git Bash (or equivalent terminal)

- At least **2 GB of free RAM** — shut down other VMs before starting

> The Vagrantfile for this section installs Docker Engine and Docker CLI automatically.


## Step 1: VM Setup

- Check and clean up any running VMs first:

```bash

vagrant global-status --prune

```

- If any VMs are running, halt them:

```bash

# Navigate to the VM's folder first

vagrant halt

```

- Navigate to the project folder and bring up the VM:

```bash

cd Devops_Labs/04-vprofile-microservices

vagrant up

```

> ⚠️ First run will take a few minutes — Docker Engine is installed automatically.

- SSH into the VM and switch to root:

```bash

vagrant ssh

sudo -i

```


## Step 2: Get the Source Code

- Clone the repository and navigate to the app folder:

```bash

git clone https://github.com/sirgeadas/Devops_Labs.git

cd Devops_Labs/04-vprofile-microservices/emartapp

```


## Step 3: Build & Run the Application

Unlike the previous container project, Docker Compose will **build the images from source** before starting the containers. 
This takes longer on first run, especially the Node.js image.

- Start all containers:

```bash

docker compose up -d

```

> ⚠️ If the build fails, try running the build step separately first:

> ```bash

> docker compose build

> docker compose up -d

> ```


- Verify all six containers are running:

```bash

docker compose ps

docker ps -a

```

> Any container in an `exited` state will be restarted automatically by Docker Compose.


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

You should see the EMart frontend served by the Angular client app via Nginx.
Run through the following checks to validate each service:

| Action | Validates |

| ------ | --------- |

| Frontend loads | Nginx + Angular container |

| Register a new user | Node.js API + MongoDB container |

| Log in successfully | Node.js API + MySQL container |

| Browse the Books section | Java Books API + MySQL container |


## Cleanup

- Stop and remove all containers:

```bash

docker compose down

```

- Remove all unused images and containers:

```bash

docker system prune -a

```

- Exit and power off the VM:

```bash

exit

vagrant halt

```

## Takeaways

This was my first hands-on look at a microservices application. A few things stood out:
- **Each service is independent** — Angular, Node.js and Java all live in their own containers, with their own databases. They only talk to each other through the API gateway
- **Docker Compose can build too** — not just pull. The `build` directive in the compose file points to a Dockerfile and compiles the image on the spot
- **Scalability makes sense now** — you could add a payment service, a cart service, a video service, each at its own endpoint, without touching anything else. That's the power of microservices
- The complexity is also real though — six containers, two databases, an API gateway. Managing this at scale is exactly what Kubernetes is for, and I can already see why 😊
