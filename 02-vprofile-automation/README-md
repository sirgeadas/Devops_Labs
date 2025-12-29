# VProfile Project – Automated Deployment

## Overview

This project demonstrates the **automated deployment** of the **VProfile Java application** using **Vagrant** and **shell provisioning scripts**.

The goal is to move beyond manual server setup and show how a **multi-tier application stack** can be reproducibly deployed using Infrastructure as Code (IaC) principles, even without advanced configuration management tools.

This project builds on the **manual deployment** by automating:
- VM provisioning
- Package installation
- Service configuration
- Application build and deployment

---

## Architecture

The application is deployed across **five virtual machines**, each with a dedicated role:

| Hostname | Role |
|--------|-----|
| `web01` | Nginx reverse proxy |
| `app01` | Tomcat application server |
| `db01` | MySQL (MariaDB) database |
| `mc01` | Memcached |
| `rmq01` | RabbitMQ message broker |

All VMs communicate over a **private network** (`192.168.56.0/24`).

---

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

---

## Vagrant Configuration

The `Vagrantfile` defines:

- Multiple VMs in a single environment
- Static private IPs
- Hostname resolution using `vagrant-hostmanager`
- Resource allocation per VM
- OS selection based on component requirements

Each service runs on its **own VM**, closely mimicking a real-world, production-like deployment.

---

## Provisioning Strategy

Each VM is provisioned using **dedicated shell scripts**, keeping responsibilities clearly separated:

### Database (`db01`)
- Installs and configures MariaDB
- Secures the database
- Creates the application database
- Imports the schema and data dump

### Memcached (`mc01`)
- Installs Memcached
- Binds to all interfaces
- Opens required firewall ports
- Enables the service on boot

### RabbitMQ (`rmq01`)
- Installs RabbitMQ and Erlang
- Creates users and permissions
- Disables loopback restrictions
- Exposes messaging ports

### Application Server (`app01`)
- Installs Java and Tomcat
- Installs Maven
- Clones the VProfile source code
- Builds the application
- Deploys the WAR file to Tomcat

### Web Server (`web01`)
- Installs Nginx
- Configures reverse proxy to Tomcat
- Enables and starts the service

---

## Deployment Flow

1. `vagrant up` is executed  
2. Vagrant creates all defined VMs  
3. Network and hostname resolution are configured  
4. Provisioning scripts run on each VM  
5. Services start automatically  
6. The application becomes accessible via the web server  

This ensures a **repeatable and predictable deployment** from a clean environment.

---

## Validation

Once provisioning completes:

- Access the application via the **Nginx server IP**
- Confirm backend services are running:
  - MySQL
  - RabbitMQ
  - Memcached
- Verify Tomcat successfully deployed the application

---

## Why This Approach

This project intentionally uses **Vagrant + shell scripts** to:

- Reinforce understanding of Linux internals
- Make automation explicit and transparent
- Avoid abstracting complexity too early
- Build strong fundamentals before moving to tools like:
  - Ansible
  - Docker
  - Kubernetes
  - Terraform

---

## Lessons Learned

- Automation exposes hidden dependencies  
- Ordering and service readiness matter  
- Reproducibility is more valuable than speed  
- Clear separation of responsibilities simplifies troubleshooting  
- Infrastructure should be disposable, not precious  

---

## Next Steps

Planned improvements and follow-ups:

- Refactor shell scripts for idempotency
- Replace shell provisioning with Ansible
- Containerize the application
- Introduce CI/CD pipelines
- Deploy to a cloud-based environment

---

## How to Run

```bash
vagrant up
```

## To destroy the environment:

```bash
vagrant destroy -f
```

This project is part of an ongoing **DevOps learning journey**, focusing on understanding systems deeply before introducing higher-level abstractions.
