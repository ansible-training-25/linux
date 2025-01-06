# 🚀 **Ansible Playbook: Web Server Deployment and Testing**

This project contains two Ansible playbooks to automate the deployment, configuration, and validation of web servers.

## 📂 **Project Structure**

```
/02-deploy-webserver
├── deploy_web.yml      # Playbook for deploying and configuring web servers
├── get_web_content.yml # Playbook for testing web server content
├── templates/
│   └── vhost.conf.j2   # Jinja2 template for virtual host configuration
└── files/
    └── index.html      # Sample web content file
```
### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/02-deploy-webserver
touch ~/02-deploy-webserver/deploy_web.yml   ~/02-deploy-webserver/get_web_content.yml ~/02-deploy-webserver/site.yml ~/02-deploy-webserver/inventory
mkdir ~/02-deploy-webserver/templates ~/02-deploy-webserver/files
touch ~/02-deploy-webserver/templates/vhost.conf.j2 
touch ~/02-deploy-webserver/files/index.html 
tree ~/02-deploy-webserver
cd ~/02-deploy-webserver
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

---
## 📝 **2. templates/vhost.conf.j2**

```jinja
<VirtualHost *:{{ http_port }}>
    ServerName {{ ansible_facts['hostname'] }}
    DocumentRoot "/var/www/vhosts/{{ ansible_facts['hostname'] }}"

    <Directory "/var/www/vhosts/{{ ansible_facts['hostname'] }}">
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog "/var/log/httpd/{{ ansible_facts['hostname'] }}_error.log"
    CustomLog "/var/log/httpd/{{ ansible_facts['hostname'] }}_access.log" combined
</VirtualHost>
```

---

## 📄 **3. files/index.html**

```html
Welcome to Ansible!

    This page is deployed and managed using Ansible.
    Happy Automation 🚀✨

```
---
## 🛠️ **1.1. deploy_web.yml (Beginner)** 

### **Description:**  
This playbook installs and configures Apache HTTP servers on managed hosts.

```yaml
---
- name: Install and configure web servers
  hosts: webservers
  become: true

  vars:
    http_port: 8080
    web_packages:
      - httpd
      - firewalld
    firewall_ports:
      - "{{ http_port }}"

  tasks:
    # Install required packages
    - name: Install required packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ web_packages }}"

    # Start required services
    - name: Start services
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop: "{{ web_packages }}"

    # Ensure configuration for the http port
    - name: "Configure http port {{ http_port }}"
      ansible.builtin.lineinfile:
        path: "/etc/httpd/conf/httpd.conf"
        regexp: '^Listen'
        line: "Listen {{ http_port }}"

    # Restart the HTTPD service
    - name: Restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted
     
    # Deploy the virtual host configuration template
    - name: Deploy configuration template
      ansible.builtin.template:
        src: templates/vhost.conf.j2
        dest: /etc/httpd/conf.d/vhost.conf
        owner: root
        group: root
        mode: '0644'

    # Restart the HTTPD service when notified
    - name: Restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted

    # Copy the index.html file to the virtual host directory
    - name: Copy index.html
      ansible.builtin.copy:
        src: files/
        dest: "/var/www/vhosts/{{ ansible_facts['hostname'] }}/"
        owner: root
        group: root
        mode: '0644'

    # Ensure the firewall allows necessary services
    - name: Ensure web server ports are open
      ansible.posix.firewalld:
        state: enabled
        permanent: true
        immediate: true
        port: "{{ item }}/tcp"
      loop: "{{ firewall_ports }}"
      when: ansible_facts['os_family'] == 'RedHat'


```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run deploy_web.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://node1.example.com:8080
curl http://node2.example.com:8080
curl http://node3.example.com:8080

```

## 🛠️ **1.2. deploy_web.yml (Advance: Handlers)** 

### **Description:**  
This playbook installs and configures Apache HTTP servers on managed hosts.

```yaml
---
- name: Install and configure web servers
  hosts: webservers
  become: true

  vars:
    http_port: 8080
    web_packages:
      - httpd
      - firewalld
    firewall_ports:
      - "{{ http_port }}"

  tasks:
    # Install required packages
    - name: Install required packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ web_packages }}"

    # Start required services
    - name: Start services
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop: "{{ web_packages }}"

    # Ensure configuration for the http port
    - name: "Configure http port {{ http_port }}"
      ansible.builtin.lineinfile:
        path: "/etc/httpd/conf/httpd.conf"
        regexp: '^Listen'
        line: "Listen {{ http_port }}"
      notify: Restart httpd
     
    # Deploy the virtual host configuration template
    - name: Deploy configuration template
      ansible.builtin.template:
        src: templates/vhost.conf.j2
        dest: /etc/httpd/conf.d/vhost.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart httpd

    # Copy the index.html file to the virtual host directory
    - name: Copy index.html
      ansible.builtin.copy:
        src: files/
        dest: "/var/www/vhosts/{{ ansible_facts['hostname'] }}/"
        owner: root
        group: root
        mode: '0644'

    # Ensure the firewall allows necessary services
    - name: Ensure web server ports are open
      ansible.posix.firewalld:
        state: enabled
        permanent: true
        immediate: true
        port: "{{ item }}/tcp"
      loop: "{{ firewall_ports }}"
      when: ansible_facts['os_family'] == 'RedHat'

  handlers:
    # Restart the HTTPD service when notified
    - name: Restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted
```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run deploy_web.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://node1.example.com:8080
curl http://node2.example.com:8080
curl http://node3.example.com:8080

```
---
## 🧪 **4.1. get_web_content.yml (Beginner)**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

```yaml
---
- name: Test web content
  hosts: localhost
  tasks:
    # Attempt to retrieve web content from target servers
    - name: Retrieve web content from node1
      ansible.builtin.uri:
        url: http://node1.example.com:8080
        return_content: true
      register: content

    # Print the returned content
    - name: Print the web content
      ansible.builtin.debug:
        msg: "{{ content }}"

    - name: Retrieve web content from node2
      ansible.builtin.uri:
        url: http://node2.example.com:8080
        return_content: true
      register: content

    # Print the returned content
    - name: Print the web content
      ansible.builtin.debug:
        msg: "{{ content }}"

    - name: Retrieve web content from node3
      ansible.builtin.uri:
        url: http://node3.example.com:8080
        return_content: true
      register: content

    # Print the returned content
    - name: Print the web content
      ansible.builtin.debug:
        msg: "{{ content }}"

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run get_web_content.yml -m stdout -i inventory 
```

## 🧪 **4.2. get_web_content.yml (Intermediate)**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

```yaml
---
- name: Test web content
  hosts: localhost
  vars:
    target_servers: 
    - node1.example.com
    - node2.example.com
    - node3.example.com
  tasks:
  # Attempt to retrieve web content from target servers
  - name: Retrieve web content
    ansible.builtin.uri:
      url: http://{{ item }}:8080
      return_content: true
    register: content
    loop: "{{ target_servers }}"
    
  # Print the returned content
  - name: Print the web content
    ansible.builtin.debug:
      msg: "{{ content }}"


```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run get_web_content.yml -m stdout -i inventory 
```
---
## 🧪 **4.3. get_web_content.yml (Advance)**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

```yaml
---
- name: Test web content
  hosts: localhost
  tasks:

    # Attempt to retrieve web content from target servers
    - name: Retrieve web content and write to error log on failure
      block:
        - name: Retrieve web content
          ansible.builtin.uri:
            url: http://{{ item }}:8080
            return_content: true
          register: content
          loop: "{{ groups['webservers'] }}"
      rescue:
        # Log error to a file if the retrieval fails
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/02-deploy-webserver/error.log
            line: "Error retrieving content"
            create: true

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run get_web_content.yml -m stdout -i inventory 
```
---

## 📋 **5. site.yml (advance)**

### **Description:**  
This master playbook imports and executes `deploy_web.yml` and `get_web_content.yml` sequentially.

```yaml
---
# Deploy and configure web servers
- name: Deploy web servers
  ansible.builtin.import_playbook: deploy_web.yml

# Validate the web servers by retrieving web content
- name: Retrieve web content
  ansible.builtin.import_playbook: get_web_content.yml
```

---

## 🚦 **Run the Playbooks**


2. **Run the playbooks using `ansible-navigator`:**
   ```bash
   ansible-navigator run site.yml -i inventory -m stdout 
   ```


---


## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
---
- name: Clean the environment
  hosts: webservers
  become: true

  vars:
    web_packages:
      - httpd
      - firewalld


  tasks:
    # Removed previously installed packages
    - name: Uninstall packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: absent
      loop: "{{ web_packages }}"

```
## 🚦 **Run the Playbooks**
   ```bash
   ansible-navigator run  playbook_clean.yml -i inventory -m stdout 
   ```
