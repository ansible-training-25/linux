# 🚀 **Ansible Playbook: Managing Users, Groups, and SSH Configuration**

This project demonstrates how to manage users, groups, sudo permissions, and SSH configurations using Ansible.

## 📂 **Project Structure**

```
/03-create-users
├── users.yml          # Playbook for managing users and authentication
├── inventory
├── vars/
│   └── users_vars.yml # Variables for users
├── files/
│   ├── user1.key.pub  # SSH public key for user1
│   ├── user2.key.pub  # SSH public key for user2
│   ├── user3.key.pub  # SSH public key for user3

```

---

### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/03-create-users
touch ~/03-create-users/users.yml  ~/03-create-users/inventory
mkdir ~/03-create-users/vars ~/03-create-users/files
touch ~/03-create-users/vars/users_vars.yml 
touch ~/03-create-users/files/user1.key.pub ~/03-create-users/files/user2.key.pub
tree ~/03-create-users
cd ~/03-create-users
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

## 🛠️ **1.1. Variables for Users and Groups**

Define users and their groups in a variable file.

**File:** `vars/users_vars.yml`

```yaml
---
users:
  - username: user1
    groups: webadmin
  - username: user2
    groups: webadmin
```

---

---

## 🛠️ **1.2. SSH Key files**

Create SSH public key files to copy them to the managed nodes.

**File:** `files/user1.key.pub`

```yaml
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDjSb8OhP5wsd+WrGl3wN0oOWPe+j8f1XxMU0FLwfI4C7+Gf4CTIsl/pF6bqAwvqUcRXGqpCHMsvgg0hHDM9cE8UqA6w176KRCU8LodMdACMOJqlXYalbgSxRkqVrbjpcJTEDPjLnaDsgbumXv8gDAOsWOpOC0uULdfOCmay0cqhbiqNqiCe1WSdrJmYMwr4ZpJNaGeokNttHc6ZvSdipIRO+ii/q1G+7zcMMonaGw3IyqMtxdtSAatm561/AxryKwJnlp8Yu2uj0yd91yHVTsX9dXS7yEvh837QTBfa9vhg8hTbh1J2gsxUl3YoblWTZk0LYthA5cKNvqSvp3xo0b79aqliV/+IpMkznr9fDOihfTNau9GYZA0UoTqh7tweoALlPMMATzI2nH0XBVm3GnXq/fk6HYFKorNYMk73pZZdmG76Sfak/n/XzuhfVDsOptLSkAFiU2ecZ3Kcvt36vOnkQalsmCZZPSSSZRMGTr/3tMPQeee6PUgWZGcq1p6G6U= 
```

**File:** `files/user2.key.pub`

```yaml
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDjSb8OhP5wsd+WrGl3wN0oOWPe+j8f1XxMU0FLwfI4C7+Gf4CTIsl/pF6bqAwvqUcRXGqpCHMsvgg0hHDM9cE8UqA6w176KRCU8LodMdACMOJqlXYalbgSxRkqVrbjpcJTEDPjLnaDsgbumXv8gDAOsWOpOC0uULdfOCmay0cqhbiqNqiCe1WSdrJmYMwr4ZpJNaGeokNttHc6ZvSdipIRO+ii/q1G+7zcMMonaGw3IyqMtxdtSAatm561/AxryKwJnlp8Yu2uj0yd91yHVTsX9dXS7yEvh837QTBfa9vhg8hTbh1J2gsxUl3YoblWTZk0LYthA5cKNvqSvp3xo0b79aqliV/+IpMkznr9fDOihfTNau9GYZA0UoTqh7tweoALlPMMATzI2nH0XBVm3GnXq/fk6HYFKorNYMk73pZZdmG76Sfak/n/XzuhfVDsOptLSkAFiU2ecZ3Kcvt36vOnkQalsmCZZPSSSZRMGTr/3tMPQeee6PUgWZGcq1p6G6U= 
```

---

## 📋 **2. Playbook: `users.yml`**

This playbook creates users, manages their SSH keys, configures sudo permissions, and ensures SSH root login is disabled.

**File:** `users.yml`

```yaml
---
- name: Create multiple local users
  hosts: infra
  become: true
  vars_files:
    - vars/users_vars.yml

  handlers:
    - name: Restart sshd
      ansible.builtin.service:
        name: sshd
        state: restarted

  tasks:
    # Ensure the webadmin group exists
    - name: Add webadmin group
      ansible.builtin.group:
        name: webadmin
        state: present

    # Create user accounts based on variables
    - name: Create user accounts
      ansible.builtin.user:
        name: "{{ item['username'] }}"
        groups: "{{ item['groups'] }}"
      loop: "{{ users }}"

    # Add SSH authorized keys for each user
    - name: Add authorized keys
      ansible.posix.authorized_key:
        user: "{{ item['username'] }}"
        key: "{{ lookup('file', 'files/' + item['username'] + '.key.pub') }}"
      loop: "{{ users }}"

    # Configure sudo to allow passwordless access for webadmin group
    - name: Modify sudo config to allow webadmin users sudo without a password
      ansible.builtin.lineinfile:
        path: /etc/sudoers.d/webadmin
        state: present
        create: true
        mode: 0440
        line: "%webadmin ALL=(ALL) NOPASSWD: ALL"
        validate: /usr/sbin/visudo -cf %s

    # Disable root login via SSH
    - name: Disable root login via SSH
      ansible.builtin.lineinfile:
        dest: /etc/ssh/sshd_config
        regexp: "^PermitRootLogin"
        line: "PermitRootLogin no"
      notify: Restart sshd
```

### **Task Explanation:**
1. **Add webadmin group:** Ensures the `webadmin` group exists.
2. **Create user accounts:** Creates users based on variables.
3. **Add SSH authorized keys:** Adds SSH keys from the `files/` directory.
4. **Modify sudo configuration:** Allows `webadmin` group passwordless sudo access.
5. **Disable root login via SSH:** Prevents direct root login.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run users.yml -m stdout -i inventory 
```

### 🧪  **Verify:**
Create a private key file to authenticate with and log in using the created users.



**File:** `private.key`

```yaml
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```



```bash
chmod 400 private.key
ssh -i private.key user1@node1 
--> user1@NODE1# sudo su -

ssh -i private.key user2@node1 
ssh -i private.key user1@node2 
ssh -i private.key user2@node2 
ssh -i private.key user1@node3 
ssh -i private.key user2@node3 

```

Try authenticating with root user, it should be disabled
```bash
ssh root@node1
ssh root@node2
ssh root@node3

```
---

---

## 📖 **Key Concepts Used**

- **User Management:** Create and manage user accounts and groups.
- **SSH Keys:** Secure SSH authentication using `ansible.posix.authorized_key`.
- **Sudo Configuration:** Enable passwordless sudo for specific groups.
- **SSH Daemon Configuration:** Secure SSH access and prevent root login.
- **Handlers:** Restart SSH daemon after configuration changes.

---

---


## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
- name: Clean up user accounts and configurations
  hosts: infra
  become: true
  vars_files:
    - vars/users_vars.yml

  handlers:
    - name: Restart sshd
      ansible.builtin.service:
        name: sshd
        state: restarted

  tasks:
    # Remove SSH authorized keys for each user
    - name: Remove authorized keys for users
      ansible.posix.authorized_key:
        user: "{{ item['username'] }}"
        key: "{{ lookup('file', 'files/' + item['username'] + '.key.pub') }}"
        state: absent
      loop: "{{ users }}"
      ignore_errors: true

    # Remove user accounts
    - name: Remove user accounts
      ansible.builtin.user:
        name: "{{ item['username'] }}"
        state: absent
        remove: true
      loop: "{{ users }}"
      ignore_errors: true

    # Remove webadmin group
    - name: Remove webadmin group
      ansible.builtin.group:
        name: webadmin
        state: absent

    # Remove sudo configuration for webadmin group
    - name: Remove sudo configuration for webadmin group
      ansible.builtin.file:
        path: /etc/sudoers.d/webadmin
        state: absent
      ignore_errors: true

    # Restore default root login setting via SSH
    - name: Enable root login via SSH
      ansible.builtin.lineinfile:
        dest: /etc/ssh/sshd_config
        regexp: "^PermitRootLogin"
        line: "PermitRootLogin yes"
      notify: Restart sshd

```
   ```bash
   ansible-navigator run playbook_clean.yml -i inventory -m stdout 
   ```

