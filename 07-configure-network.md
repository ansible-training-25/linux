# 🚀 **Ansible Playbook: RHEL system networking confiouration and testing**

This project contains an Ansible playbooks to use a system collection to Adjust the network configuration of a managed nodes.

## 📂 **Project Structure**

```
/07-configure-network
├── configure-network.yml       # Playbook for deploying and configuring database servers
├── get-network.yml             #Playbook to obtain network configuration 
├── inventory


```
### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/07-configure-network
touch ~/07-configure-network/configure-network.yml  ~/07-configure-network/get-network.yml ~/07-configure-network/inventory
mkdir ~/07-configure-network/collections
tree ~/07-configure-network/
cd ~/07-configure-network
```
### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[infra]
node3

```
---

## 🛠️ **1. Install system network Ansible Collection**

### **Install my_namespace.my_collection  Collection Directly**

Use the `ansible-galaxy` command to install the `my_namespace.my_collection ` collection directly from a tarball.

```bash
ls ~/artifacts/redhat-rhel_system_roles-1.19.3.tar.gz
ansible-galaxy collection install ~/artifacts/redhat-rhel_system_roles-1.19.3.tar.gz -p collections/
tree
```

### **Verify Installed Collections**

```bash
ansible-galaxy collection list -p collections
```

**Expected Output:**
```
Collection               Version
------------------------ -------
redhat.rhel_system_roles 1.19.3 
```

---

## 🛠️ **1.1. get-network.yml**

### **Description:**  
This playbook gets the eth0 primary ip address of managed hosts.

```yaml
---
- name: Obtain network info for infra servers
  hosts: infra
  gather_facts: yes

  tasks:

    - name: Display eth0 info
      ansible.builtin.debug:
        var: ansible_facts['eth0']['ipv4']
```
### **Run the get playbook**
```bash
ansible-navigator run get-network.yml -i inventory -m stdout 
```

**Expected Output:**
```
TASK [Display eth0 info] *******************************************************
ok: [node3] => {
    "ansible_facts['eth0']['ipv4']": {
        "address": "172.16.9.69",
        "broadcast": "172.16.255.255",
        "netmask": "255.255.0.0",
        "network": "172.16.0.0",
        "prefix": "16"
    }

```

## 🛠️ **1.2. get-network.yml**

### **Description:**  
This playbook gets the eth0 primary and secondary ip addresses of managed hosts.

```yaml
---
- name: Obtain network info for infra servers
  hosts: infra
  gather_facts: yes

  tasks:

    - name: Display eth0 primary info
      ansible.builtin.debug:
        msg: "The primary IP is: {{ ansible_facts['eth0']['ipv4']['address'] }}"


    - name: Display eth0 secondary info
      ansible.builtin.debug:
        msg: "The first secondary IP is: {{ ansible_facts['eth0']['ipv4_secondaries'][0]['address'] }}"
      when:  ansible_facts['eth0']['ipv4_secondaries'] is defined
```


**Expected Output:**
```
TASK [Gathering Facts] *********************************************************
ok: [node3]

TASK [Display eth0 primary info] ***********************************************
ok: [node3] => {
    "msg": "The primary IP is: 172.16.219.219 \n"
}

TASK [Display eth0 secondary info] *********************************************
skipping: [node3]



```



### **Run the get Playbook**
```bash
ansible-navigator run get-network.yml -i inventory -m stdout 
```

## 🛠️ **2. configure-network.yml **

### **Description:**  
This playbook installs and configures networking on managed hosts.

```yaml
---
- name: Network Connection Configuration
  hosts: infra
  become: true
  vars:
    network_connections:
      - name: eth0
        type: ethernet
        state: up
        ip:
          address:
            - "{{ ansible_facts['eth0']['ipv4']['address'] }}/{{ ansible_facts['eth0']['ipv4']['prefix'] }}"
            - 192.168.1.100/24
  roles:
    - redhat.rhel_system_roles.network

```

### **Run the Playbook**
```bash
ansible-navigator run configure-network.yml -i inventory -m stdout 
```
### 🚦  **1- Verify: Ensure the network configuration**
Execute the following command to verify that the network configuration were applied to the nodes.

```bash
ssh node3 ip address show eth0
```

### 🚦 **2- Verify using Ansible: Run the get playbook**
```bash
ansible-navigator run get-network.yml -i inventory -m stdout 
```
**Expected Output:**
```
TASK [Display eth0 primary info] ***********************************************
ok: [node3] => {
    "msg": "The primary IP is: 172.16.9.69"
}

TASK [Display eth0 secondary info] *********************************************
ok: [node3] => {
    "msg": "The first secondary IP is: 192.168.1.100"
}


```

