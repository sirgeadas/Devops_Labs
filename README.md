# DevOps Labs

A hands-on DevOps learning journey — from manual server setup to containers, cloud, and beyond.

Each folder in this repo documents a real lab: the architecture, the steps, the commands, and what I took away from it. This is both a study journal and a portfolio of practical work.

## Labs

| # | Project | Description | Stack |
|---|---------|-------------|-------|
| 01 | [VProfile – Manual Deployment](./01-vprofile-manual/README.md) | Multi-tier app deployed by hand across 5 VMs | Vagrant, Nginx, Tomcat, MySQL, RabbitMQ, Memcached |
| 02 | [VProfile – Automated Deployment](./02-vprofile-automation/README.md) | Same stack, fully automated with shell provisioning scripts | Vagrant, Bash |
| 03 | [VProfile – Containers Intro](./03-vprofile-containers/README.md) | Entire stack running on a single VM with one command | Docker, Docker Compose |
| 04 | [VProfile – Microservices](./04-vprofile-microservices/README.md) | Microservices e-commerce app with an API Gateway pattern | Docker, Docker Compose |

## Roadmap

This repo will grow as I progress through the following topics:

- [x] Manual provisioning
- [x] Automated provisioning with Vagrant + Bash
- [x] Containers intro with Docker Compose
- [ ] Bash Scripting
- [ ] AI-assisted Scripting
- [ ] AWS – Lift & Shift
- [ ] CI/CD with Jenkins
- [ ] GitHub Actions
- [ ] GitLab CI
- [ ] Terraform
- [ ] Ansible
- [ ] Monitoring & Observability
- [ ] AWS Part 2
- [ ] Google Cloud
- [ ] Docker & Containerization (deep dive)
- [ ] Kubernetes
- [ ] App Deployment on Kubernetes
- [ ] GitOps

## Tools & Technologies

A running list of everything covered across these labs:

`Vagrant` `VirtualBox` `Bash` `Nginx` `Apache Tomcat` `MySQL` `MariaDB` `RabbitMQ` `Memcached` `Maven` `Java` `Docker` `Docker Compose`

## How to Use This Repo

Each lab folder is self-contained. Every `README.md` includes the architecture, prerequisites, step-by-step instructions, and personal takeaways. Clone the repo and follow along:

```bash
git clone https://github.com/sirgeadas/Devops_Labs.git
```

Then navigate to whichever lab you want to explore.
