# 🚀 **Ansible Role: vhost_role - Apache Virtual Host Deployment**

This project contains an Ansible role named `vhost_role` to automate the deployment, configuration, and validation of an Apache web server with a virtual host.

## 📂 **Project Structure**

```
/04-vhost-role
├── call-vhost-role.yml   # Playbook using the vhost_role role
├── inventory
├── roles/
│   ├── vhost_role/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── vhost.conf.j2
│   │   ├── files/
│   │   │   ├── html/
│   │   │   │   └── index.html
```
### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/04-vhost-role
touch ~/04-vhost-role/call-vhost-role.yml  ~/04-vhost-role/inventory
mkdir ~/04-vhost-role/roles
ansible-galaxy init vhost_role --init-path ~/04-vhost-role/roles

touch ~/04-vhost-role/roles/vhost_role/templates/vhost.conf.j2 
touch ~/04-vhost-role/roles/vhost_role/files/index.html
tree ~/04-vhost-role
cd ~/04-vhost-role
```
### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[webservers]
node1
node2
node3


```

---

## 🛠️ **1. roles/vhost_role**

### **Description:**  
This role installs and configures the Apache HTTP server with a virtual host.


**Template File:** `roles/vhost_role/templates/vhost.conf.j2`

```apache
# {{ ansible_managed }}

<VirtualHost *:{{ http_port }}>
    ServerAdmin webmaster@{{ ansible_fqdn }}
    ServerName {{ ansible_fqdn }}
    ErrorLog logs/{{ ansible_hostname }}-error.log
    CustomLog logs/{{ ansible_hostname }}-common.log common
    DocumentRoot /var/www/vhosts/{{ ansible_hostname }}/

    <Directory /var/www/vhosts/{{ ansible_hostname }}/>
        Options +Indexes +FollowSymlinks +Includes
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

**Files:** `roles/vhost_role/files/index.html`

```
Welcome to Ansible!

    This page is deployed and managed using Ansible.
    Happy Automation 🚀✨
```

**Vars File:** `roles/vhost_role/defaults/main.yml`

```yaml
---
http_port: 80

```


**Tasks File:** `roles/vhost_role/tasks/main.yml`

```yaml
---
# Ensure the Apache package is installed
- name: Ensure httpd is installed
  ansible.builtin.dnf:
    name: httpd
    state: latest

# Ensure the Apache service is enabled and started
- name: Ensure httpd is started and enabled
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

# Ensure configuration for the http port
- name: "Configure http port {{ http_port }}"
  ansible.builtin.lineinfile:
    path: "/etc/httpd/conf/httpd.conf"
    regexp: '^Listen'
    line: "Listen {{ http_port }}"
  notify:
    - restart httpd

# Deploy the virtual host configuration
- name: vhost file is rendered
  ansible.builtin.template:
    src: vhost.conf.j2
    dest: /etc/httpd/conf.d/vhost.conf
    owner: root
    group: root
    mode: '0644'
  notify:
    - restart httpd

# Copy the index.html file to the virtual host directory
- name: Copy index.html
  ansible.builtin.copy:
    src: index.html
    dest: "/var/www/vhosts/{{ ansible_facts['hostname'] }}/"
    owner: root
    group: root


#Deal with Firewall block
- name: Manage firewalld stuff
  block:
  # Check if firewalld is running on the system
  - name: Check if firewalld is running
    ansible.builtin.service_facts:

  # Ensure the firewall allows necessary services
  - name: Ensure web server ports are open
    ansible.posix.firewalld:
      state: enabled
      permanent: true
      immediate: true
      port: "{{ http_port }}/tcp"
    when: 
    - "'firewalld.service' in ansible_facts.services"
    - ansible_facts.services['firewalld.service'].state == 'running'

      
```

**Handlers File:** `roles/vhost_role/handlers/main.yml`

```yaml
---
# Restart Apache service when configuration changes
- name: restart httpd
  ansible.builtin.service:
    name: httpd
    state: restarted
```

---

## 📋 **2.1 call-vhost-role.yml (Beginner) **

### **Description:**  
This playbook uses the `vhost_role` role to configure and deploy the Apache virtual host and ensures web content is copied.

**Master playbook:**   `call-vhost-role.yml`

```yaml
---
- name: Use vhost_role role playbook
  hosts: webservers
  become: true
  tasks:
  - name: Include vhost role
    ansible.builtin.include_role:
      name: vhost_role

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run call-vhost-role.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://node1:8080
curl http://node2:8080
curl http://node3:8080

```

## 📋 **2.2 call-vhost-role.yml (Advance) **

### **Description:**  
This playbook uses the `vhost_role` role to configure and deploy the Apache virtual host and ensures web content is copied.

**Master playbook:**   `call-vhost-role.yml`

```yaml
---
- name: Use vhost_role role playbook
  hosts: webservers
  become: true

  pre_tasks:
    - name: pre_tasks message
      ansible.builtin.debug:
        msg: 'Ensure web server configuration.'

  roles:
    - vhost_role

  post_tasks:
    - name: post_tasks message
      ansible.builtin.debug:
        msg: 'Web server is configured.'
```
---
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run call-vhost-role.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://node1:8080
curl http://node2:8080
curl http://node3:8080

```
**Order of execution:**
- pre tasks
- roles
- tasks
- handlers notified
- post tasks
---

## 📖 **Key Concepts Used**

- **Roles:** Structure tasks, handlers, files, and templates.
- **Variables:** Dynamic configurations in templates.
- **Handlers:** Restart services on configuration changes.
- **Templates:** Jinja2 for virtual host configurations.
- **File Management:** Copy static content.

---



## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
---
- name: Clean the environment
  hosts: all
  become: true

  vars:
    packages:
      - httpd


  tasks:
    # Removed previously installed packages
    - name: Uninstall packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: absent
      loop: "{{ packages }}"

```
   ```bash
   ansible-navigator run playbook_clean.yml -i inventory -m stdout 
   ```

