# step1: Develop an ansible module for installation and configuration of PostgreSQL instance to a Linux server


- To develop an Ansible module for the installation and configuration of a PostgreSQL instance on a Linux server, follow these steps:

- Create a new directory for the module in your Ansible playbook directory

- Create a new file named postgresql_module.py in the roles/postgresql_module/ directory.
#  Ansible-01: To Install Ansible and Ansible Basic Operations

The purpose of this hands-on training is to give  knowledge of basic Ansible skills.

## Learning Outcomes

At the end of this hands-on training, students will be able to;

- Explain what Ansible can do
- Learn basic Ad-hoc commands  

## Outline

- Part 1 - Install Ansible



## Part 1 - Install Ansible


- Spin-up 3 Amazon Linux 2 instances and name them as:
    
     node1 ----> (SSH PORT 22, HTTP PORT 80)


```bash
sudo yum update -y
sudo amazon-linux-extras install ansible2
```

### Confirm Installation

- To confirm the successful installation of Ansible, run the following command.

```bash
$ ansible --version
```

### Configure Ansible on AWS EC2

- Connect to the control node and for this basic inventory, edit /etc/ansible/hosts, and add a few remote systems (manage nodes) to the end of the file. For this example, use the IP addresses of the servers.

```bash
$ sudo su
$ cd /etc/ansible
$ ls
$ vim hosts
```
```bash
[webservers]
node1 ansible_host=<node1_ip> ansible_user=ec2-user



```bash
 mkdir roles/playbook.yaml/
```





 

# step1a:  Create a playbook that uses the new module to install and configure PostgreSQL on a Linux server.




```yaml
---
- hosts: postgresql-servers
  become: true

  tasks:
    - name: Install PostgreSQL
      yum:
        name: postgresql-server
        state: present

    - name: Initialize PostgreSQL database
      become_user: postgres
      become_method: su
      command: initdb -D /var/lib/pgsql/data

    - name: Start PostgreSQL service
      systemd:
        name: postgresql
        state: started

    - name: Enable PostgreSQL service on boot
      systemd:
        name: postgresql
        enabled: true

    - name: Set PostgreSQL password for user postgres
      postgresql_user:
        db: postgres
        user: postgres
        password: "{{ postgres_password }}"

    - name: Create PostgreSQL database
      postgresql_db:
        name: "{{ postgres_db }}"
        owner: "{{ postgres_user }}"
        encoding: UTF8
        lc_collate: "en_US.UTF-8"
        lc_ctype: "en_US.UTF-8"

    - name: Configure PostgreSQL to listen on all interfaces
      lineinfile:
        path: /var/lib/pgsql/data/postgresql.conf
        regexp: '^#?listen_addresses = .*'
        line: 'listen_addresses = '*''

    - name: Configure PostgreSQL authentication
      lineinfile:
        path: /var/lib/pgsql/data/pg_hba.conf
        line: 'host    all             all             0.0.0.0/0               md5'

    - name: Restart PostgreSQL service
      systemd:
        name: postgresql
        state: restarted

 ```       



- Run the playbook.

```bash

  ansible-playbook playbook.yaml
```


# step2  Develop a bash script for installation and configuration of PostgreSQL instance to a Linux server



*** This script does the following:

-Adds the PostgreSQL repository to the list of available repositories
-Installs PostgreSQL
-Starts the PostgreSQL service
-Creates a new PostgreSQL user and database
-Configures PostgreSQL to allow connections from any IP address
-Restarts the PostgreSQL service to apply the changes


```bash



 #!/bin/bash

# Add PostgreSQL repository
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update

# Install PostgreSQL
sudo apt-get install -y postgresql

# Start PostgreSQL service
sudo systemctl start postgresql

# Create a new PostgreSQL user and database
sudo -u postgres psql -c "CREATE USER dbuser WITH ENCRYPTED PASSWORD 'dbpass123';"
sudo -u postgres psql -c "CREATE DATABASE dbname OWNER dbuser;"

# Configure PostgreSQL to allow connections from any IP address
sudo sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/g" /etc/postgresql/$(ls /etc/postgresql)/main/postgresql.conf
echo "host    all             all             0.0.0.0/0               md5" | sudo tee -a /etc/postgresql/$(ls /etc/postgresql)/main/pg_hba.conf

# Restart PostgreSQL service to apply changes
sudo systemctl restart postgresql

```


