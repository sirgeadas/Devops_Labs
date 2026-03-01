# VProfile Project – Automated Deployment

> 📁 Part 2 of my DevOps learning path — previous: [Manual Deployment](../01-vprofile-manual/README.md) | next: [Containers Intro](../03-vprofile-containers/README.md)

## Overview

Same VProfile stack as before, but this time I'm not SSH'ing into five VMs and typing commands by hand. 
The goal here was to automate the entire deployment using **Vagrant** and **shell provisioning scripts**, so the whole environment comes up with a single command.

This builds directly on the manual deployment. Everything I configured by hand in Part 1 is now scripted and reproducible.

## Architecture

Five VMs, each with a dedicated role, communicating over a private network (`192.168.56.0/24`):

| Hostname | Service | Role |
| -------- | ------- | ---- |
| `web01` | Nginx | Reverse proxy |
| `app01` | Tomcat | Java application server |
| `db01` | MySQL (MariaDB) | Relational database |
| `mc01` | Memcached | Caching layer |
| `rmq01` | RabbitMQ | Message broker |

## Technology Stack

- **Vagrant** – VM orchestration
- **VirtualBox** – Virtualization provider
- **Shell scripting** – Provisioning and configuration
- **Nginx** – Reverse proxy
- **Apache Tomcat** – Java application server
- **MySQL (MariaDB)** – Database backend
- **RabbitMQ** – Messaging service
- **Memcached** – Caching layer
- **Maven** – Application build tool
- **Java** – Application runtime

## Prerequisites

- [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)
- Vagrant Hostmanager plugin:

```bash
vagrant plugin install vagrant-hostmanager
```
- Git Bash (or equivalent terminal)

## How It Works

The `Vagrantfile` defines all five VMs: Static IPs, hostnames, resource allocation and OS.
Also points each one to a dedicated shell script that handles its own provisioning.

### Database (`db01`)
- Installs and configures MariaDB
- Secures the installation
- Creates the application database and imports the schema

### Memcached (`mc01`)
- Installs Memcached and binds it to all interfaces
- Opens required firewall ports and enables on boot

### RabbitMQ (`rmq01`)
- Installs RabbitMQ
- Creates users, sets permissions, disables loopback restrictions

### Application Server (`app01`)
- Installs Java, Tomcat and Maven
- Clones the VProfile source, builds it and deploys the WAR file

### Web Server (`web01`)
- Installs Nginx and configures the reverse proxy to Tomcat

## Deployment Flow

1. `vagrant up` is executed
2. Vagrant creates all five VMs
3. Network and hostname resolution are configured automatically
4. Each VM runs its provisioning script
5. Services start and the app becomes accessible

## How to Run

- Clone the repository:
```bash
git clone https://github.com/sirgeadas/Devops_Labs.git
```

- Navigate to the 02-vprofile-automation directory:
```bash
cd Devops_Labs/02-vprofile-automation
```

- Bring up the environment:
```bash
vagrant up
```

> ⚠️ First run will take a while. Each VM is provisioned from scratch.


## Validation

- Get the Nginx VM IP and open it in the browser:
```
http://192.168.56.11
```

- Log in with:
  - **Username:** `admin_vp`
  - **Password:** `admin_vp`

- Verify backend services are running on each VM:
```bash
vagrant ssh <hostname>
sudo systemctl status <service>
```

## Cleanup

- Destroy all VMs:
```bash
vagrant destroy -f
```

## Takeaways

Doing this right after the manual deployment made the value of automation immediately obvious. 
Every service dependency, startup order and configuration detail I had struggled with manually was now a script.
And the whole thing came up without me touching a single VM!

A few things that stuck with me:

- Order still matters. The scripts need to account for service dependencies just like the manual steps did
- Infrastructure should be **disposable**. If something breaks, `vagrant destroy -f && vagrant up` is a full reset
- This approach has limits though. Shell scripts aren't idempotent by default, and they don't scale well. That's what tools like Ansible, Docker and Kubernetes are for.
- And that's exactly where this journey is heading. 😊
