# VProfile Project – Manual Deployment

> 📁 Part 1 of my DevOps learning path — next: [Automated Deployment](../02-vprofile-automation/README.md)

## Overview

This is where it all started. Before automating anything, I wanted to understand every moving part of the stack by setting it all up by hand — SSH'ing into each VM, installing services, configuring them, and wiring everything together manually.

It's tedious, but that's the point. You can't appreciate automation until you've done it the hard way first.

## Architecture

The VProfile app is a Java-based social networking application. It runs across five VMs, each dedicated to one service:

| Hostname | Service | Role |
| -------- | ------- | ---- |
| `web01` | Nginx | Web server / reverse proxy |
| `app01` | Tomcat | Java application server |
| `db01` | MySQL (MariaDB) | Relational database |
| `mc01` | Memcached | Database caching |
| `rmq01` | RabbitMQ | Message broker |

Services must be provisioned in this order due to dependencies:

1. MySQL ➡️ Database
2. Memcached ➡️ Caching layer
3. RabbitMQ ➡️ Message broker
4. Tomcat ➡️ Application server
5. Nginx ➡️ Web server / load balancing

## Prerequisites

- [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)
- Vagrant Hostmanager plugin:
```bash
vagrant plugin install vagrant-hostmanager
```
- Git Bash (or equivalent terminal)

> **Optional:** IDE or text editor (VS Code, Sublime Text, Notepad++)

## Step 1: VM Setup

- Clone the repository:
```bash
git clone https://github.com/sirgeadas/Devops_Labs.git
```

- Navigate to the manual provisioning directory:
```bash
cd Devops_Labs/01-vprofile-manual
```

- Bring up all five VMs:
```bash
vagrant up
```

> ⚠️ This may take a while. If it stops mid-way, run `vagrant up` again. Hostnames and `/etc/hosts` entries are updated automatically by the hostmanager plugin.

## Step 2: Provisioning Services

### 2.1 MySQL Setup

- SSH into the DB VM:
```bash
vagrant ssh db01
```

- Verify `/etc/hosts` entries:
```bash
cat /etc/hosts
```

- Update OS and install required packages:
```bash
sudo dnf update -y
sudo dnf install epel-release git mariadb-server -y
```

- Start and enable MariaDB:
```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

- Secure the installation:
```bash
sudo mysql_secure_installation
```

Follow the prompts as below:

```
Set root password? [Y/n] Y
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] N
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

- Create the application database and user:
```mysql
mysql -u root -padmin123
CREATE DATABASE accounts;
GRANT ALL PRIVILEGES ON accounts.* TO 'admin'@'localhost' IDENTIFIED BY 'admin123';
GRANT ALL PRIVILEGES ON accounts.* TO 'admin'@'%' IDENTIFIED BY 'admin123';
FLUSH PRIVILEGES;
EXIT;
```

- Import the database schema:
```bash
cd /tmp/
git clone https://github.com/sirgeadas/Devops_Labs.git
cd Devops_Labs/01-vprofile-manual
mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
```

- Configure firewall and restart MariaDB:
```bash
sudo systemctl restart mariadb
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
sudo systemctl restart mariadb
```

### 2.2 Memcached Setup

- SSH into the Memcached VM:
```bash
vagrant ssh mc01
```

- Update OS and install Memcached:
```bash
sudo dnf update -y
sudo dnf install epel-release memcached -y
sudo systemctl start memcached
sudo systemctl enable memcached
```

- Configure Memcached to listen on all interfaces:
```bash
sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
sudo systemctl restart memcached
```

- Configure firewall:
```bash
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --add-port=11211/tcp --permanent
sudo firewall-cmd --reload
```

### 2.3 RabbitMQ Setup

- SSH into the RabbitMQ VM:
```bash
vagrant ssh rmq01
```

- Update OS and install RabbitMQ:
```bash
sudo dnf update -y
sudo dnf install epel-release wget -y
sudo dnf -y install centos-release-rabbitmq-38
sudo dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
sudo systemctl enable --now rabbitmq-server
```

- Configure the admin user:
```bash
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
sudo rabbitmqctl add_user test test
sudo rabbitmqctl set_user_tags test administrator
sudo rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
sudo systemctl restart rabbitmq-server
```

- Configure firewall:
```bash
sudo firewall-cmd --add-port=5672/tcp --permanent
sudo firewall-cmd --reload
```

### 2.4 Tomcat Setup

- SSH into the App VM:
```bash
vagrant ssh app01
```

- Update OS and install dependencies:
```bash
sudo dnf update -y
sudo dnf install epel-release java-17-openjdk java-17-openjdk-devel unzip git wget -y
```

- Download and install Tomcat:
```bash
cd /tmp/
wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
tar xzvf apache-tomcat-10.1.26.tar.gz
sudo useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
sudo cp -r apache-tomcat-10.1.26/* /usr/local/tomcat/
sudo chown -R tomcat.tomcat /usr/local/tomcat
```

- Create the systemd service file:
```bash
sudo vi /etc/systemd/system/tomcat.service
```

```ini
[Unit]
Description=Tomcat
After=network.target

[Service]
User=tomcat
Group=tomcat
WorkingDirectory=/usr/local/tomcat
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/var/tomcat/%i/run/tomcat.pid
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINA_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

- Reload systemd and start Tomcat:
```bash
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl enable tomcat
```

- Configure firewall:
```bash
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

### 2.5 Build & Deploy the Application

- Install Maven:
```bash
cd /tmp/
wget https://archive.apache.org/dist/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.zip
unzip apache-maven-3.9.9-bin.zip
sudo cp -r apache-maven-3.9.9 /usr/local/maven3.9
export MAVEN_OPTS="-Xmx512m"
```

- Clone the source code and update the application config:
```bash
git clone https://github.com/sirgeadas/Devops_Labs.git
cd Devops_Labs/01-vprofile-manual
vi src/main/resources/application.properties
```

- Build and deploy:
```bash
/usr/local/maven3.9/bin/mvn install
sudo systemctl stop tomcat
sudo rm -rf /usr/local/tomcat/webapps/ROOT*
sudo cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
sudo chown -R tomcat.tomcat /usr/local/tomcat/webapps
sudo systemctl start tomcat
```

### 2.6 Nginx Setup

- SSH into the Web VM and switch to root:
```bash
vagrant ssh web01
sudo -i
```

- Update OS and install Nginx:
```bash
apt update && apt upgrade -y
apt install nginx -y
```

- Create the virtual host config:
```bash
vi /etc/nginx/sites-available/vproapp
```

```nginx
upstream vproapp {
    server app01:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://vproapp;
    }
}
```

- Enable the site and restart Nginx:
```bash
rm -rf /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
systemctl restart nginx
```

## Validation

- Open a browser and navigate to:
```
http://192.168.56.11
```

- Log in with:
  - **Username:** `admin_vp`
  - **Password:** `admin_vp`

- Verify each service is running on its VM:
```bash
vagrant ssh <hostname>
sudo systemctl status <service>
```

## Cleanup

- To halt all VMs:
```bash
vagrant halt
```

- To destroy all VMs:
```bash
vagrant destroy -f
```

## Takeaways

Going through this manually was genuinely valuable. Every firewall rule, every systemd unit, every config file — I had to understand it well enough to type it myself. Nothing was hidden behind a script.

A few things that stuck with me:

- Service **order matters** — Tomcat needs the database and cache to be ready before it starts
- Each service has its own quirks — MariaDB needs securing, Memcached needs to be told to listen on all interfaces, RabbitMQ needs loopback restrictions lifted
- Doing this by hand once makes you deeply appreciate what automation does for you — which is exactly what Part 2 is about 😊
