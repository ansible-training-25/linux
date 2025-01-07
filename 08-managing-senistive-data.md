# 🚀 **Ansible Playbook: Managing sensitive data**


This project includes an Ansible playbook designed to create users, securely handle user passwords as sensitive data, and encrypt them.

## 📂 **Project Structure**

```
/08-managing-senistive-data
├── create_users.yml      # Playbook for creating users
├── secret.yml            # Vault encrypted data
├── key.txt               # Vault encryption key
├── inventory
```
### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/08-managing-senistive-data
touch ~/08-managing-senistive-data/create_users.yml ~/08-managing-senistive-data/key.txt  ~/08-managing-senistive-data/inventory
tree ~/08-managing-senistive-data
cd ~/08-managing-senistive-data
```

### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[infra]
node1
node2
node3


```
---
## 🛠️ ** secret.yml ** 

### **Description:**  
This file conatins the encrypted user password

```bash
ansible-vault create secret.yml
New Vault password: devoteam
Confirm New Vault password: devoteam

```
use your desired encryption key

declare username and password variables
**--(press 'a' character inside the editor to edit)--**
```txt
username: myvault_user
password: myvault_password
```
save thefile
**--(press ESC and then :wq) to save and exit--**

## 🛠️ ** create_users.yml ** 

### **Description:**  
This playbook creates system users and ensure that password authentication is enabled

```yaml
---
- name: Create user accounts 
  hosts: infra
  become: true
  vars_files:
    - secret.yml
  tasks:
    - name: Creating user from secret.yml
      ansible.builtin.user:
        name: "{{ username }}"
        password: "{{ password | password_hash('sha512') }}"
      
    - name: Ensure SSH password authentication is enabled
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#PasswordAuthentication'
        line: 'PasswordAuthentication yes'
        state: present

    - name: Restart SSH service to apply changes
      service:
        name: sshd
        state: restarted
```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run create_users.yml --pae false -m stdout -i inventory --vault-id @prompt
```


## 🛠️ ** key.txt ** 

### **Description:**  
This file conatins the encryption key as plain text

```txt
devoteam
```
### 🚦 **Run the Playbook with key file instead of prompts:**
```bash
ansible-navigator run create_users.yml -m stdout -i inventory --vault-password-file=key.txt
```



### 🚦  **Verify:**
Ensure you can login to the nodes with myvault_user
```bash
ssh myvault_user@node1
ssh myvault_user@node2
ssh myvault_user@node3

```
