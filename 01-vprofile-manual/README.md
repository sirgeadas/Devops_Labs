# VProfile Project – Manual Deployment

This project is the **first step in my DevOps career path**, demonstrating how to manually set up a multi-tier web application stack locally.

The goal here is hands-on understanding of each service, before automating everything in the next phase using Vagrant and scripts.

## Project Overview

The VProfile Project is a simplified social networking web application built in **Java**.

The stack consists of:

|       Service |                           Role |
| ------------: | -----------------------------: |
|         Nginx |     Web server / Load balancer |
|        Tomcat |        Java application server |
|      RabbitMQ | Message broker / Queuing agent |
|     Memcached |               Database caching |
| ElasticSearch |      Indexing / Search service |
|         MySQL |            Relational database |

**Deployment Flow:**
1. MSQL           ➡️ Database
2. Memcached ➡️ Database caching
3. RabbitMQ    ➡️ Message broker
4. Tomcat         ➡️ Application server
5. Nginx           ➡️ Web server / load balancing

> Manual deployment ensures you understand how each component works, the dependencies, and the order of setup.

------
## Prerequisites

Make sure the following are installed on your host machine:
- [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)
- Vagrant Hostmanager plugin:
```bash
vagrant plugin install vagrant-hostmanager
```
- Git Bash (or equivalent terminal)

> **Optional:** IDE or text editor (VS Code, Sublime Text, Notepad++)

### Step 1: VM Setup

- Clone the repository:
```bash
git clone -b local https://github.com/hkhcoder/vprofile-project.git
```

- Navigate to the manual provisioning directory:
```bash
cd vprofile-project/vagrant/Manual_provisioning
```

- Bring up the VMs:
```bash
vagrant up
```

>⚠️**Note:** Depending on your system, this may take a while. If it stops mid-way, run vagrant up again.
>The hostnames and /etc/hosts entries are updated automatically.

### Step 2: Provisioning Services

The services need to be set up in order as per Deployment Flow.
#### 2.1 MySQL Setup

- SSH into the DB VM:
```bash
vagrant ssh db01
```
- Verify /etc/hosts entries and update if needed:
```bash
cat /etc/hosts
```
- Update OS and install required packages:
```bash
sudo dnf update -y
```
```bash
sudo dnf install epel-release git mariadb-server -y
```
- Start and enable MariaDB:
```bash
sudo systemctl start mariadb
```
```bash
sudo systemctl enable mariadb
```
- Secure MariaDB installation:
```bash
sudo mysql_secure_installation
```
- Set root password (admin123 for this tutorial)

- Remove anonymous users

- Remove test database

- Keep remote root login (optional)

- Reload privileges

**Example:**
```
Set root password? [Y/n] **Y**
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
... Success!
By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them. This is intended only for testing, and to make the installation
go a bit smoother. You should remove them before moving into a
production environment.
Remove anonymous users? [Y/n] **Y**
... Success!
Normally, root should only be allowed to connect from 'localhost'. This
ensures that someone cannot guess at the root password from the network.
Disallow root login remotely? [Y/n] **N**
... skipping.
By default, MariaDB comes with a database named 'test' that anyone can
access. This is also intended only for testing, and should be removed
before moving into a production environment.
Remove test database and access to it? [Y/n] **Y**
- Dropping test database...
... Success!
- Removing privileges on test database...
... Success!
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
Reload privilege tables now? [Y/n] **Y**
... Success!
```

- Create database and user:
```mysql
mysql -u root -padmin123
CREATE DATABASE accounts;
GRANT ALL PRIVILEGES ON accounts.* TO 'admin'@'localhost' IDENTIFIED BY 'admin123';
GRANT ALL PRIVILEGES ON accounts.* TO 'admin'@'%' IDENTIFIED BY 'admin123';
FLUSH PRIVILEGES;
EXIT;
```

- Initialize database from source code:
```bash
cd /tmp/
```
```bash
git clone -b local https://github.com/hkhcoder/vprofile-project.git
```
```bash
cd vprofile-project
```
```bash
mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
```

- Restart MariaDB and configure firewall:
```bash
sudo systemctl restart mariadb
```
```bash
sudo systemctl start firewalld
```
```bash
sudo systemctl enable firewalld
```
```bash
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
```
```bash
sudo firewall-cmd --reload
```
```bash
sudo systemctl restart mariadb
```
#### 2.2 Memcached Setup

- SSH into Memcache VM:
```bash
vagrant ssh mc01
```

- Verify /etc/hosts entries.

- Update OS and install Memcached:
```bash
sudo dnf update -y
```
```bash
sudo dnf install epel-release memcached -y
```
```bash
sudo systemctl start memcached
```
```bash
sudo systemctl enable memcached
```
```bash
sudo systemctl status memcached
```

- Configure Memcached to listen on all interfaces:
```bash
sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
```
```bash
sudo systemctl restart memcached
```

- Configure firewall:
```bash
sudo systemctl start firewalld
```
```bash
sudo systemctl enable firewalld
```
```bash
sudo firewall-cmd --add-port=11211/tcp --permanent
```
```bash
sudo firewall-cmd --reload
```

#### 2.3 RabbitMQ Setup

- SSH into RabbitMQ VM:
```bash
vagrant ssh rmq01
```

- Verify /etc/hosts, update if needed.

- Update OS, enable EPEL repo, and install RabbitMQ:
```bash
sudo dnf update -y
```
```bash
sudo dnf install epel-release wget -y
```
```bash
sudo dnf -y install centos-release-rabbitmq-38
```
```bash
sudo dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
```
```bash
sudo systemctl enable --now rabbitmq-server
```

- Configure admin user:
```bash
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
```
```bash
sudo rabbitmqctl add_user test test
````
```bash
sudo rabbitmqctl set_user_tags test administrator
```
```bash
sudo rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
```
```bash
sudo systemctl restart rabbitmq-server
```

- Configure firewall:
```bash
sudo firewall-cmd --add-port=5672/tcp --permanent
```
```bash
sudo firewall-cmd --reload
```

#### 2.4 Tomcat Setup

- SSH into App VM:
```bash
vagrant ssh app01
```

- Verify /etc/hosts, update if needed.

- Update OS and install dependencies:
```bash
sudo dnf update -y
```
```bash
sudo dnf install epel-release java-17-openjdk java-17-openjdk-devel unzip git wget -y
```

- Download and install Tomcat:
```bash
cd /tmp/
```
```bash
wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
```
```bash
tar xzvf apache-tomcat-10.1.26.tar.gz
```
```bash
sudo useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
```
```bash
sudo cp -r apache-tomcat-10.1.26/* /usr/local/tomcat/
```
```bash
sudo chown -R tomcat.tomcat /usr/local/tomcat
```

- Configure Tomcat as a systemd service (/etc/systemd/system/tomcat.service):
```bash
sudo vi /etc/systemd/system/tomcat.service
```

```bash
[Unit]
Description=Tomcat
After=network.target
```
```bash
[Service]
User=tomcat
Group=tomcat
WorkingDirectory=/usr/local/tomcat
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/var/tomcat/%i/run/tomcat.pid
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINE_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
RestartSec=10
Restart=always
```
```bash
[Install]
WantedBy=multi-user.target
```

- Reload systemd and start Tomcat:
```bash
sudo systemctl daemon-reload
```
```bash
sudo systemctl start tomcat
```
```bash
sudo systemctl enable tomcat
```

- Configure firewall:
```bash
sudo firewall-cmd --add-port=8080/tcp --permanent
```
```bash
sudo firewall-cmd --reload
```

#### 2.5 Build & Deploy Application (Tomcat)

- Install Maven:
```bash
cd /tmp/
```
```bash
wget https://archive.apache.org/dist/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.zip
```
```bash
unzip apache-maven-3.9.9-bin.zip
```
```bash
sudo cp -r apache-maven-3.9.9 /usr/local/maven3.9
```
```bash
export MAVEN_OPTS="-Xmx512m"
```
- Clone source code and update configuration:
```bash
git clone -b local https://github.com/hkhcoder/vprofile-project.git
```
```bash
cd vprofile-project
```
```bash
vi src/main/resources/application.properties
```

- Build and deploy the application:
```bash
/usr/local/maven3.9/bin/mvn install
```
```bash
sudo systemctl stop tomcat
```
```bash
sudo rm -rf /usr/local/tomcat/webapps/ROOT*
```
```bash
sudo cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
```
```bash
sudo chown -R tomcat.tomcat /usr/local/tomcat/webapps
```
```bash
sudo systemctl start tomcat
```
```bash
sudo systemctl restart tomcat
```

#### 2.6 Nginx Setup

- SSH into Web VM:
```bash
vagrant ssh web01
```
```bash
sudo -i
```

- Update OS and install Nginx:
```bash
apt update && apt upgrade -y
```
```bash
apt install nginx -y
```

- Configure Nginx (/etc/nginx/sites-available/vproapp):
```bash
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

- Enable site and restart Nginx:
```bash
rm -rf /etc/nginx/sites-enabled/default
```
```bash
ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
```bash
systemctl restart nginx
```

## Validation

Open a browser and navigate to the Nginx VM IP.

You should see the VProfile web application running.

Verify services individually (Tomcat, MySQL, Memcached, RabbitMQ) using systemctl or service commands.

##### Notes:

This is manual deployment — next phase will automate all VMs and services using Vagrant and bash provisioning scripts.

Manual setup is a learning step: it builds understanding of service dependencies, order of setup, and configuration challenges.

Firewalls are configured for required ports; in production environments, more strict rules would apply.
