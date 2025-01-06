# 🚀 **Ansible Playbooks: Simple File Operations with Copy and Lineinfile Modules**

This project contains three simple Ansible playbooks demonstrating the use of the `copy` and `lineinfile` modules.

## 📂 **Project Structure**

```
/01-simple-copy
├── playbook1.yml   # Write a simple line to a file
├── playbook2.yml   # Write a line with system facts
├── playbook3.yml   # Replace a specific string in a file
├── inventory       # Inventory file for target hosts
```

---
### **Login as student into ansible-1 control node:**
```bash
sudo su  - student
mkdir ~/01-simple-copy
touch ~/01-simple-copy/playbook1.yml ~/01-simple-copy/playbook2.yml ~/01-simple-copy/playbook3.yml ~/01-simple-copy/inventory
tree ~/01-simple-copy
cd ~/01-simple-copy
```

### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[group1]
node1
node2

[group2]
node2
node3

```

## 🛠️ **1. Playbook 1: Write a Simple Line to a File**

**Playbook:** `playbook1.yml`

```yaml
---
- name: Write a simple line to a file
  hosts: group1 #only hosts of GROUP "group1"
  become: true

  tasks:
    - name: Write a static line to a file
      ansible.builtin.copy:
        dest: /tmp/simple_file.txt
        content: "This is a static line written by Ansible."
        owner: root
        group: root
        mode: '0644'
```

### **Explanation:**
- Creates a file `/tmp/simple_file.txt`.
- Writes a static line to the file.
- Ensures the correct ownership and permissions.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run  playbook1.yml -m stdout -i inventory
```
### 🚦 **Verify:**
```bash
ssh node1 cat /tmp/simple_file.txt
ssh node2 cat /tmp/simple_file.txt
```
---

## 🧪 **2. Playbook 2: Write a Line with System Facts**

**Playbook:** `playbook2.yml`

```yaml
---
- name: Write system facts to a file
  hosts: group2 #only hosts of GROUP "group2"
  become: true

  tasks:
    - name: Write Memory and IP information to a file
      ansible.builtin.copy:
        dest: /tmp/facts_file.txt
        content: |
          Memory: {{ ansible_memtotal_mb }} MB
          IP Address: {{ ansible_default_ipv4.address }}
        owner: root
        group: root
        mode: '0644'
```

### **Explanation:**
- Gathers system facts (`ansible_memtotal_mb`, `ansible_default_ipv4.address`).
- Writes Memory, and IP Address information to `/tmp/facts_file.txt`.
- Ensures proper ownership and permissions.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run playbook2.yml -m stdout -i inventory 
```
### 🚦 **Verify:**
```bash
ssh node2 cat /tmp/facts_file.txt
ssh node3 cat /tmp/facts_file.txt
```
---

## ✏️ **3. Playbook 3: Replace a Specific String in a File**

**Playbook:** `playbook3.yml`

```yaml
---
- name: Replace a specific string in a file
  hosts: all,!node3
  become: true

  tasks:
    - name: Replace 'static' with 'dynamic' in the file
      ansible.builtin.lineinfile:
        path: /tmp/simple_file.txt
        regexp: 'static'
        line: 'This is a dynamic line written by Ansible.'
```

### **Explanation:**
- Searches for the word `static` in `/tmp/simple_file.txt`.
- Replaces it with the word `dynamic`.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run playbook3.yml -m stdout -i inventory 
```
### 🚦**Verify:**
```bash
ssh node1 cat /tmp/simple_file.txt
ssh node2 cat /tmp/simple_file.txt
```

---

## ✅ **Expected Results**

1. **Playbook 1:** Creates `/tmp/simple_file.txt` with a static line.
2. **Playbook 2:** Creates `/tmp/facts_file.txt` with system facts.
3. **Playbook 3:** Updates `/tmp/simple_file.txt`, replacing `static` with `dynamic`.

---

## 📖 **Key Concepts Used**

- **Copy Module:** Write static or dynamic content to files.
- **System Facts:** Retrieve CPU, memory, and IP information.
- **Lineinfile Module:** Replace or modify specific lines in files.

---

## 🚦 **4. Clean Up the Environment**

After testing, clean up the environment:
**Playbook:** `playbook_clean.yml`
```yaml
---
- name: Clean nodes
  hosts: all
  become: true

  tasks:
    - name: Remove previously created files
      ansible.builtin.file:
        path: "{{ item }}"
        state: absent
      loop:
      - /tmp/simple_file.txt
      - /tmp/facts_file.txt
```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run playbook_clean.yml -m stdout -i inventory 
```
---

### 🚦 **Verify:**
```bash
ssh node1 ls /tmp/simple_file.txt /tmp/facts_file.txt
ssh node2 ls /tmp/simple_file.txt /tmp/facts_file.txt
ssh node3 ls /tmp/simple_file.txt /tmp/facts_file.txt

```
### **Explanation:**
- **/tmp/simple_file.txt** and **/tmp/facts_file.txt** should not be exist anymore.
