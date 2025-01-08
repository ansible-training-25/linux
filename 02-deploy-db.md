# 🚀 **Ansible Playbook: Database Server Deployment and Testing**

This project contains two Ansible playbooks to automate the deployment, configuration, and validation of database servers.

## 📂 **Project Structure**

```
/02-deploy-db
├── db_deploy.yml       # Playbook for deploying and configuring database servers
├── templates/
│   └── db_config.cnf.j2 # Jinja2 template for database configuration
    └── init_db.sql.j2    # SQL script for initializing database

```
### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/02-deploy-db
touch ~/02-deploy-db/db_deploy.yml  ~/02-deploy-db/inventory
mkdir ~/02-deploy-db/templates/
touch ~/02-deploy-db/templates/db_config.cnf.j2 ~/02-deploy-db/templates/init_db.sql.j2
cd ~/02-deploy-db
```
### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[db_servers]
node1
node2
node3


```
---
## 📄 **templates/db_config.cnf.j2**

```jinja2
[mysqld]
user=mysql
bind-address=0.0.0.0
port={{ db_port }}
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Performance and tuning
max_connections=200
innodb_buffer_pool_size=1G
innodb_log_file_size=256M

[client]
port={{ db_port }}
socket=/var/lib/mysql/mysql.sock
```

## 📄 **templates/init_db.sql.j2**

```sql
-- Create a sample database
CREATE DATABASE IF NOT EXISTS {{database.name }};

-- Switch to the sample database
USE {{database.name }};

-- Create a sample table
CREATE TABLE IF NOT EXISTS my_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(50) NOT NULL,
    category VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO my_table (product_name, category) 
VALUES 
    ('Laptop', 'Electronics'),
    ('Office Chair', 'Furniture');

-- Create a dedicated database users
CREATE USER IF NOT EXISTS '{{ database.user }}'@'%' IDENTIFIED BY '{{ database.password }}';
GRANT ALL PRIVILEGES ON {{database.name }}.* TO '{{ database.user }}'@'%';
FLUSH PRIVILEGES;
```

## 🛠️ **1. db_deploy.yml**

### **Description:**  
This playbook installs and configures MySQL database servers on managed hosts.

```yaml
---
- name: Install and configure database servers
  hosts: db_servers
  become: true

  vars:
    db_packages:
      - mysql-server
      - mysql
    db_service: mysqld
    db_config_file: /etc/my.cnf
    db_port: 3306
    database:
      name: mydb
      user: myuser
      password: mypassword
  tasks:
    # Install required database packages
    - name: Install database packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ db_packages }}"

    # Tune the directory permissions for mysqld
    - name: Tune the permissions
      ansible.builtin.file:
        path: /var/log
        mode: '0777'
        recurse: true

    # Start and enable database service
    - name: Start and enable database service
      ansible.builtin.service:
        name: "{{ db_service }}"
        state: started
        enabled: true

    # Deploy database configuration template
    - name: Deploy database configuration
      ansible.builtin.template:
        src: templates/db_config.cnf.j2
        dest: "{{ db_config_file }}"
        owner: root
        group: root
        mode: '0644'
      notify: Restart database service

    #Deal with Firewall block
    - name: Manage firewalld stuff
      block:
      # Check if firewalld is running on the system
      - name: Check if firewalld is running
        ansible.builtin.service_facts:

      # Ensure the firewall allows necessary services
      - name: Ensure DB server ports are open
        ansible.posix.firewalld:
          state: enabled
          permanent: true
          immediate: true
          port: "{{ db_port }}/tcp"
        when: 
        - "'firewalld.service' in ansible_facts.services"
        - ansible_facts.services['firewalld.service'].state == 'running'

    # Render initialization template
    - name: init_db.sql file is rendered
      ansible.builtin.template:
        src: init_db.sql.j2
        dest: /tmp/init_db.sql

    # Initialize database with SQL script
    - name: Initialize database schema
      ansible.builtin.shell:
        cmd: "mysql < /tmp/init_db.sql"
        creates: /var/lib/mysql/initialized

  handlers:
    # Restart the MySQL service when notified
    - name: Restart database service
      ansible.builtin.service:
        name: "{{ db_service }}"
        state: restarted
```

### **Run the Playbook**
```bash
ansible-navigator run db_deploy.yml -i inventory -m stdout 
```
### 🚦  **Verify: Ensure the Loadbalanacing**
Execute the following command to verify that the DB repond to queries.

```bash

ssh node1 
mysql -u myuser -p 
USE mydb;
SELECT * FROM my_table;




```

**Explanation:**
- `CREATE DATABASE`: Ensures a database named `{{ database.name }}` exists.
- `CREATE USER`: Sets up a database user (`{{ database.user }}`) with privileges on `{{database.name }}`.
- `GRANT`: Grants all necessary permissions to the user.
- `FLUSH PRIVILEGES`: Ensures privilege changes take effect.
---

## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
---
- name: Clean the environment
  hosts: db_servers
  become: true

  vars:
    db_packages:
      - mysql-server
      - mysql

  tasks:
    # Removed previously installed packages
    - name: Uninstall packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: absent
      loop: "{{ db_packages }}"


      

```

