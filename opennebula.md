# Installing OpenNebula on Proxmox

## Summary

1. [First Steps](#1-first-steps)<br>
   1.1. [Creating a VM](#11-creating-a-vm)<br>
   1.2. [Creating a user](#12-creating-a-user)<br>
   1.3. [Installing MariaDB](#13-installing-mariadb)<br>

2. [Configuring the Repositories](#2-configuring-the-repositories)<br>
   2.1. [Enabling Dependencies](#21-enabling-dependencies)<br>
   2.2. [Adding OpenNebula Repository](#22-adding-opennebula-repository)<br>

3. [Single Front-end Installation](#3-single-front-end-installation)<br>
   3.1. [Adding Third Party Repositories](#31-adding-third-party-repositories)<br>
   3.2. [Installing OpenNebula Front-end](#32-installing-opennebula-front-end)<br>
   3.3. [Installing Dependencies for OpenNebula Edge Clusters Provisioning](#33-installing-dependencies-for-opennebula-edge-clusters-provisioning)<br>
   3.4. [Enabling MariaDB](#34-enabling-mariadb)<br>
   3.5. [OneGate Server](#35-onegate-server)<br>
   3.6. [OneFlow](#36-oneflow)<br>
   3.7. [Starting OpenNebula and Configuring the Firewall](#37-starting-opennebula-and-configuring-the-firewall)<br>
   3.8. [Accessing the Front-end](#38-accessing-the-front-end)<br>
4. [References](#4-references)

---

## 1. First Steps

### 1.1. Creating a VM

- Create a VM with the following configuration:
  - OS: Rocky Linux 9.4 (minimal)
  - 4 processors (1 socket, 4 cores) [host]
  - Memory: 8 GiB

> Leave the rest as default.

- Start the VM

> Open the console to check if it has started properly.

- SSH into the VM

- Check the VM IP using

```bash
ip a
```

- In your local terminal, enter

```bash
ssh root@<VM-IP>
```

- Update the system

```bash
dnf update
```

### 1.2. Creating a user

- It is recommended to create an user instead of using the root user
- To create a new user, enter

```bash
useradd -m -s /bin/bash <username>
passwd <username>
usermod -aG wheel <username>    # add the new user to the wheel group, so they have administrative privileges
```

- Flags explained:

  - `-m` automatically creates the directory `/home/<username>`
  - `-s /bin/bash` defines `Bash` as the default shell
  - `-a` appends the user to a group without removing them from the previous groups
  - `-G` specifies the group list to which the new user will be appended to

- SSH into the new user

```bash
ssh <username>@<VM-IP>
```

### 1.3. Installing MariaDB

```bash
sudo dnf install mariadb-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo mariadb-secure-installation
```

> In the following questions, add a password, remove the anonymous user and the test table.

- Enter MariaDB

```bash
sudo mysql -u root -p
```

- Inside MariaDB, create the user `oneadmin`

```sql
CREATE USER 'oneadmin' IDENTIFIED BY '<db-password>';
GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

- Exit MariaDB using `CTRL + D` or typing `exit;`

---

## 2. Configuring the Repositories

### 2.1. Enabling Dependencies

- For Rocky Linux, we are going to use the RHEL Community Edition repositories

- In rhel9 and AlmaLinux9 some dependencies cannot be found in the default repositories. Some extra repositories need to be enabled. To do this, execute the following

```bash
sudo su

repo=$(yum repolist --disabled | grep -i -e powertools -e crb | awk '{print $1}' | head -1) # find out the disabled repositories names

yum config-manager --set-enabled $repo  # enable the repo

yum makecache   # update packages cache
```

### 2.2. Adding OpenNebula Repository

- Create a repository file to allow Rocky Linux to install OpenNebula's packages directly from the official website

```bash
cat << "EOT" > /etc/yum.repos.d/opennebula.repo
[opennebula]
name=OpenNebula Community Edition
baseurl=https://downloads.opennebula.io/repo/7.0/RedHat/$releasever/$basearch
enabled=1
gpgkey=https://downloads.opennebula.io/repo/repo2.key
gpgcheck=1
repo_gpgcheck=1
EOT

yum makecache   # update packages cache
```

---

## 3. Single Front-end Installation

### 3.1. Adding Third Party Repositories

```bash
sudo rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

- Enable CRB repository

```bash
sudo /usr/bin/crb enable
```

### 3.2. Installing OpenNebula Front-end

```bash
sudo yum -y install opennebula opennebula-fireedge opennebula-gate opennebula-flow
```

- You can check the installation and the status

```bash
which oned

systemctl status opennebula
```

### 3.3. Installing dependencies for OpenNebula Edge Clusters provisioning

```bash
sudo su

curl 'https://releases.hashicorp.com/terraform/0.14.7/terraform_0.14.7_linux_amd64.zip' | zcat >/usr/bin/terraform

chmod 0755 /usr/bin/terraform
```

### 3.4. Enabling MariaDB

- Before you run OpenNebula for the first time, you’ll need to set the database backend and connection details in the configuration file `/etc/one/oned.conf` as follows:

```conf
# Sample configuration for MySQL
DB = [ BACKEND = "mysql",
       SERVER  = "localhost",       # IP/hostname
       PORT    = 0,                 # 0 to use the default port
       USER    = "oneadmin",        # MariaDB user
       PASSWD  = "<db-password>",   # MariaDB password
       DB_NAME = "opennebula",
       CONNECTIONS = 25,
       COMPARE_BINARY = "no" ]
```

- Log in as oneadmin user

```bash
sudo -u oneadmin /bin/sh
```

- Create file `/var/lib/one/.one/one_auth` with initial password in the format `oneadmin:<password>`

```bash
echo 'oneadmin:<password>' > /var/lib/one/.one/one_auth
```

> This will set the oneadmin’s password only upon starting OpenNebula for the first time. From that point, you must use the oneuser passwd command to change oneadmin’s password.

### 3.5. OneGate Server

- The OneGate server allows communication between VMs and OpenNebula.

- Configure it as follows

```bash
sudo vim /etc/one/onegate-server.conf
```

- Update the parameter with service listening address accordingly - use 0.0.0.0 to work on all configured network interfaces on the Front-end

```conf
:host: 0.0.0.0
```

- Configure OpenNebula Daemon

```bash
sudo vim /etc/one/oned.conf
```

- Set the `ONEGATE_ENDPOINT` with the URL:PORT of your OneGateServer. The endpoint address must be reachable directly from your future Virtual Machines.

```conf
ONEGATE_ENDPOINT = "<http://<VM-IP>:5030>"
```

### 3.6. OneFlow

- The OneFlow server orchestrates the services and multi-VM deployments.

- Configure it as follows

```bash
sudo vim /etc/one/oneflow-server.conf
```

- Update the parameter with service listening address accordingly - use 0.0.0.0 to work on all configured network interfaces on the Front-end

```conf
:host: 0.0.0.0
```

### 3.7. Starting OpenNebula and Configuring the Firewall

```bash
systemctl start opennebula opennebula-fireedge opennebula-gate opennebula-flow

systemctl enable opennebula opennebula-fireedge opennebula-gate opennebula-flow
```

- Open all [firewall ports used by OpenNebula](https://docs.opennebula.io/6.10/installation_and_configuration/frontend_installation/install.html#frontend-fw)

```bash
sudo firewall-cmd --permanent --add-port=2474/tcp
sudo firewall-cmd --permanent --add-port=2616/tcp
sudo firewall-cmd --permanent --add-port=2633/tcp
sudo firewall-cmd --permanent --add-port=4124/tcp
sudo firewall-cmd --permanent --add-port=4124/udp
sudo firewall-cmd --permanent --add-port=5030/tcp
sudo firewall-cmd --permanent --add-port=29876/tcp
sudo firewall-cmd --permanent --add-port=5900-5999/tcp
sudo firewall-cmd --permanent --add-port=49152-49215/tcp
sudo firewall-cmd --reload
```

- The list below shows the ports used by OpenNebula. These ports need to be open for OpenNebula to work properly:

|    Port     | Description                                                   |
| :---------: | ------------------------------------------------------------- |
|     22      | Front-end host SSH server                                     |
|    2474     | OneFlow server                                                |
|    2616     | Next-generation GUI server FireEdge                           |
|    2633     | Main OpenNebula Daemon (oned), XML-RPC API endpoint           |
|    4124     | Monitoring daemon (both TCP/UDP)                              |
|    5030     | OneGate server                                                |
|    29876    | noVNC Proxy Server                                            |
|    5900+    | VNC Server ports on Hosts for VMs. See VNC_PORTS              |
| 49152-49215 | Host-Host port communication required for KVM live migrations |

- Restart the server

```bash
sudo reboot now
```

- To verify the installation, enter `oneadmin` system user

```bash
sudo -u oneadmin /bin/sh
```

- Run the command

```sh
oneuser show
```

- The output should be something similar to

```sh
USER 0 INFORMATION
ID              : 0
NAME            : oneadmin
GROUP           : oneadmin
PASSWORD        : 3bc15c8aae3e4124dd409035f32ea2fd6835efc9
AUTH_DRIVER     : core
ENABLED         : Yes

USER TEMPLATE
TOKEN_PASSWORD="ec21d27e2fe4f9ed08a396cbd47b08b8e0a4ca3c"

RESOURCE USAGE & QUOTAS
```

### 3.8. Accessing the Front-end

- In your browser, enter the URL

```
http://<frontend_address>:2616/fireedge/sunstone
```

- You should see OpenNebula's front-end

![img01](/assets-opennebula/img01.png)

- Log in using `oneadmin` credentials

```
Username: oneadmin
Password: </var/lib/one/.one/one_auth>
```

## 4. References

- [Official OpenNebula Documentation](https://docs.opennebula.io/7.0/software/installation_process/manual_installation/)
