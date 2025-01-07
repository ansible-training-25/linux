# 🚀 **Ansible Role: vhost_role - Apache Virtual Host Deployment with Molecule Testing**

This project contains an Ansible role named `vhost_role` to automate the deployment, configuration, and validation of an Apache web server with a virtual host, including Molecule tests using the Podman driver.

---

## 📂 **Project Structure (without Molecule)**

```plaintext
/09-molecule-tests
├── call-vhost-role.yml   # Playbook using the vhost_role role
├── inventory             # Inventory file defining target hosts
├── roles/
│   ├── vhost_role/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── vhost.conf.j2
│   │   ├── files/
│   │   │   └── html/
│   │   │       └── index.html

```

## 🚦 **Setup Instructions**

### **Login and Create Project Directory:**

```bash
sudo su - student
mkdir ~/09-molecule-tests
cp -rf ~/solutions/04-vhost-role/* ~/09-molecule-tests
tree ~/09-molecule-tests
cd ~/09-molecule-tests
```

---

## 🛠️ **Ansible Role: vhost_role**

### **Role Description:**
As shown before, this vhost role installs and configures the Apache HTTP server with a virtual host.

## 📋 **2. Adding Molecule Testing**

### **Add Molecule to your role**

```bash
cd ~/09-molecule-tests/roles/vhost_role
molecule init scenario 
tree
```


## 📂 **Project Structure (with Molecule)**

```plaintext
/09-molecule-tests
├── call-vhost-role.yml   # Playbook using the vhost_role role
├── inventory             # Inventory file defining target hosts
├── roles/
│   ├── vhost_role/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml

        +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
        ├── molecule/             # Molecule testing directory
        │   └── default/          # Default scenario for Molecule
        │       ├── converge.yml  # Molecule playbook to test role
        │       ├── molecule.yml  # Molecule configuration file
        │       └── verify.yml    # Playbook to verify the test
        +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

│   │   ├── templates/
│   │   │   └── vhost.conf.j2
│   │   ├── files/
│   │   │   └── html/
│   │   │       └── index.html
```

---


### 🚦 **Molecule Configuration:** `molecule/default/molecule.yml`

```yaml
---
dependency:
  name: galaxy
driver:
  name: podman

platforms:
  - name: testing-instance
    image: registry.access.redhat.com/ubi9/ubi
    pre_build_image: true



scenario:
  name: default
  test_sequence:
    - destroy
    - create
    - prepare
    - converge
    - verify
    - destroy

```

### 🚦 **Molecule Creation:** `molecule/default/create.yml`

```yaml
---
- name: Create Podman instances
  hosts: localhost
  tasks:
    - name: Create Podman container
      containers.podman.podman_container:
        name: testing-instance
        image: "registry.access.redhat.com/ubi9/ubi"
        state: started
        command: /sbin/init
        privileged: true
        volumes:
          - /sys/fs/cgroup:/sys/fs/cgroup:ro
```
### 🚦 **Molecule destruction:** `molecule/default/destroy.yml`

```yaml
---
- name: Destroy Podman instance
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Destroy Podman container
      containers.podman.podman_container:
        name: testing-instance
        state: absent
```


 🚦 **Molecule destruction:** 
Try to create and destroy the testing infra using molecule, verify the creation and destruction of the container.
```bash
molecule create
podman ps
molecule destroy
podman ps
```

### 🚦 **Molecule Preparation:** `molecule/default/prepare.yml`

```yaml
---
- name: Prepare
  hosts: all
  gather_facts: true
  tasks:
    - name: Give warning message to all users upon logging
      ansible.builtin.lineinfile:
        path: /root/.bashrc
        line: "echo The server is currently being used for testing purposes. PLEASE DONT TOUCH!!"
  
```

Try to create and destroy the testing infra using molecule, verify the creation and destruction of the container.
```bash
molecule create
podman ps
molecule prepare
```

**Verify preparation step:** 
Attempt to log in to the container to verify that the message appears upon login.

```bash
podman exec -it testing-instance /bin/bash
```
**Expected Output:**
```
The server is currently being used for testing purposes. PLEASE DONT TOUCH!!
[root@dd43d1265804 /]# 
```
ensure you logged out fromt he container and get back to the control node

```bash
[root@dd43d1265804 /]# exit
```
**Expected Output:**
```
[student@ansible-1 vhost_role]$
```



### **Molecule Converge Playbook:** `molecule/default/converge.yml`

```yaml
---
- name: Converge
  hosts: all
  gather_facts: true
  vars:
    http_port: 9090
  roles:
    - vhost_role
```

---

### 🚦 **Run Molecule converge:**

```bash
molecule converge
```

---


### **Molecule Verify Playbook:** `molecule/default/verify.yml`

```yaml
---
- name: Verify role functionality
  hosts: all
  gather_facts: false
  vars:
    http_port: 9090
  tasks:
    - name: Check Apache service is running
      ansible.builtin.shell: systemctl is-active httpd
      register: service_status
    - name: Fail if Apache is not running
      ansible.builtin.fail:
        msg: "Apache is not running!"
      when: service_status.stdout != "active"


    - name: Hitting the web server
      ansible.builtin.shell: "curl localhost:{{ http_port }}"
      register: content

    - name: Fail if the server doesn't give the desired response 
      ansible.builtin.fail:
        msg: "The server is not behaving as expected!"
      when: '"Welcome to Ansible" not in content.stdout'

    - name: Printing the returned response
      ansible.builtin.debug:
        msg: "The returned response is: {{ content.stdout }}"


    
```

---

### 🚦 **Run Molecule verify:**

```bash
molecule verify
```

---


### 🚦 **Run Molecule destroy:**

```bash
molecule destroy
```

---
### 🚦 **Experiment with Molecule steps individually:**

```bash
molecule create
molecule prepare
molecule converge
molecule verify
molecule destroy

```

### 🚦 **Run all Molecule test steps together in one step (test):**

```bash
molecule reset
molecule test
```
