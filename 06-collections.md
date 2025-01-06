# 🚀 **Ansible Playbook: Using Roles and Modules from Content Collections**

This project demonstrates how to install Ansible Content Collections, use roles, and leverage modules from those collections.

## 📂 **Project Structure**

```
/06-collections
├── backup.yml             # Playbook using my_namespace.my_collection  collection
├── inventory
├── collections/
│   ├── gls/utils/
```
### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/06-collections
touch ~/06-collections/backup.yml  ~/06-collections/inventory
mkdir ~/06-collections/collections

tree ~/06-collections
cd ~/06-collections
```
### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[group1]
node1

[group2]
node2
node3



```
---

## 🛠️ **1. Install Ansible Collections**

### **Install my_namespace.my_collection  Collection Directly**

Use the `ansible-galaxy` command to install the `my_namespace.my_collection ` collection directly from a tarball.

```bash
ls ~/artifacts/my_namespace-my_collection-1.0.0.tar.gz
ansible-galaxy collection install ~/artifacts/my_namespace-my_collection-1.0.0.tar.gz -p collections/
```

### **Verify Installed Collections**

```bash
ansible-galaxy collection list -p collections
```

**Expected Output:**
```
Collection               Version
------------------------ -------
my_namespace.my_collection               1.0.0
```

---

## 📋 **2. Backup Configuration Using my_namespace.my_collection  Collection**

**Playbook:** `backup.yml`

```yaml
---
- name: Backup the system configuration
  hosts: all
  become: true
  vars:
    check_string: Hello Ansible
  tasks:
    # Check if a string has a capital letter
    - name: Check if string has capital letters
      my_namespace.my_collection.check_capital_letter:
        string: "{{ check_string }}"
      register: result

    - name: Print the result
      ansible.builtin.debug:
        msg: "{{ result.message }}"

    # Backup configuration files with my_namespace.my_collection.backup role
    - name: Ensure configuration files are saved
      ansible.builtin.include_role:
        name: my_namespace.my_collection.backup
      vars:
        backup_dir: /var/ansible-backup
        backup_id: backup_etc
        backup_files:
          - /etc/sysconfig
          - /etc/yum.repos.d
```


### **Run the Playbook**
```bash
ansible-navigator run backup.yml -i inventory -m stdout -e check_string=hellothere
```
### 🚦  **Verify: Ensure the Loadbalanacing**
Execute the following command to verify that the backup was already taken.

```bash

ssh node1 sudo ls -l /var/ansible-backup
ssh node2 sudo ls -l /var/ansible-backup
ssh node3 sudo ls -l /var/ansible-backup


```
**Expected response:** '-rw-r--r--. 1 root root 8324 Jun  27 14:17 **backup-backup_etc.tgz**'
---

## 📖 **Key Concepts Used**

- **Ansible Galaxy:** Install and manage collections with `ansible-galaxy`.
- **Modules:** Use `my_namespace.my_collection.newping` for connectivity tests.
- **Roles:** Utilize `my_namespace.my_collection.backup` 


---




