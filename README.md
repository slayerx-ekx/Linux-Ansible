# **ANSIBLE CONFIGURATION ON UBUNTU LTS jammy**

### Ansible Administration 1

### Quiz-1 : Playbook

**Create the working directory and necessary files**

```
mkdir quiz-2
cd quiz-2
touch ansible.cfg inventory quiz-2_variables.yml
```
**Define ansible.cfg**

```
[defaults]
inventory = ./inventory
```
**Create the inventory**

```
[pod]
pod-dett-managed2
```

**Create the playbook with variables**

```
- name: Quiz Playbook
  hosts: pod-dett-managed2
  remote_user: student
  become: true
  tasks:
    - name: Install latest version of apache2
      apt:
        name: apache2
        state: latest

    - name: Install latest version of mariadb-server
      apt:
        name: mariadb-server
        state: latest

    - name: Install latest version of php
      apt:
        name: php
        state: latest

    - name: Install latest version of php-mysql
      apt:
        name: php-mysql
        state: latest

    - name: Ensure apache2 service is enabled and running
      service:
        name: apache2
        state: started
        enabled: true

    - name: Ensure mariadb service is enabled and running
      service:
        name: mariadb
        state: started
        enabled: true

    - name: Generate web content for testing
      copy:
        content: "Adinusa quiz Playbook - dett"
        dest: /var/www/html/index.php

- name: Test webserver access from control node
  hosts: localhost
  tasks:
    - name: Test access to the apache2 web server
      uri:
        url: http://pod-dett-managed2/index.php
        return_content: no
        status_code: 200

```

**Run With Sintax Check**

```
ansible-playbook --syntax-check quiz-1_playbook.yml
ansible-playbook quiz-1_playbook.yml
```

**Verification**

*make sure it's outside the folder when checking or verifying*

```
ls -l quiz-1
```

**Check if the packages are installed on the managed hosts**

```
ansible pod-dett-managed2 -m shell -a "dpkg -l | grep -E 'apache2|mariadb-server|php|php-mysql'"
```

**Verify that the apache2 and mariadb services are running**

```
ansible pod-dett-managed2 -m shell -a "systemctl status apache2 | grep 'active (running)'"
ansible pod-dett-managed2 -m shell -a "systemctl status mariadb | grep 'active (running)'"

```

**Confirm the existence of the /var/www/html/index.php file with the correct content**

```
ansible pod-username-managed2 -m shell -a "cat /var/www/html/index.php"
```

**Test webserver access**
```
curl http://pod-dett-managed2/index.php
```

### Ansible Administration 1

### Quiz-2 : Variables

**Create the working directory and necessary files**

```
mkdir quiz-2
cd quiz-2
touch ansible.cfg inventory quiz-2_variables.yml
```

**Define ansible.cfg**

```
[defaults]
inventory = ./inventory
```

**Create the inventory**

```
[pod]
pod-dett-managed2
```

**Create the playbook with variables**

```
- name: Use Variables in Playbook
  hosts: pod-dett-managed2
  remote_user: student
  become: true
  vars:
    required_Pkg:
      - apache2
      - python3-urllib3
    web_Service: apache2
    content_File: "adinusa lab quiz variables - dett"
    dest_File: /var/www/html/index.html
  tasks:
    - name: Install required packages
      apt:
        name: "{{ item }}"
        state: latest
      loop: "{{ required_Pkg }}"
      
    - name: Ensure web service is enabled and running
      service:
        name: "{{ web_Service }}"
        state: started
        enabled: true

    - name: Ensure specific content exists
      copy:
        content: "{{ content_File }}"
        dest: "{{ dest_File }}"

- name: Test webserver access from control node
  hosts: localhost
  tasks:
    - name: Test access to the web server
      uri:
        url: http://pod-dett-managed2/index.html
        return_content: no
        status_code: 200
```

**Run With Sintax Check**

```
ansible-playbook --syntax-check quiz-1_playbook.yml
ansible-playbook quiz-1_playbook.yml
```

**Verification**

*make sure it's outside the folder when checking or verifying*

```
ls -l quiz-1
```

**Check if the packages are installed on the managed host**

```
ansible pod-dett-managed2 -m shell -a "dpkg -l | grep -E 'apache2|python3-urllib3'"
```

##Verify that the apache2 service is running:**

```
ansible pod-dett-managed2 -m shell -a "systemctl status apache2 | grep 'active (running)'"
```

**Confirm the existence of the /var/www/html/index.html**

```
ansible pod-dett-managed2 -m shell -a "cat /var/www/html/index.html"
```

**Test webserver**

```
curl http://pod-dett-managed2/index.html
```

### Ansible Administration 1

### Quiz-3 : Jinja 2 template

```
mkdir ~/quiz-3
cd ~/quiz-3
touch ansible.cfg inventory nginx.list.j2 mariadb.list.j2 quiz-3_j2template.yml
```

**Define ansible.cfg**

```
[defaults]
inventory = ./inventory
```

**Create the inventory**

```
[webservers]
pod-dett-managed1
pod-dett-managed2
```

**Create the Jinja2 templates 'nginx.list.j2'**

```
deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/mainline/ubuntu jammy nginx
deb-src [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/mainline/ubuntu jammy nginx
```

**Create the Jinja2 templates 'mariadb.list.j2**

```
deb https://archive.mariadb.org/mariadb-10.9/repo/ubuntu/ jammy main
deb-src https://archive.mariadb.org/mariadb-10.9/repo/ubuntu/ jammy main
```
**Create the playbook 'quiz-3_j2template.yml'**

```
- name: Quiz Jinja 2
  hosts: all
  become: true
  vars:
    required_pkg:
      - nginx=1.23.1-1~jammy
      - mariadb-server
      - mariadb-client
  tasks:
    - name: Copy Nginx Repo
      template:
        src: nginx.list.j2
        dest: /etc/apt/sources.list.d/nginx.list


    - name: Install GPG Key for Nginx Repo
      shell: |
        curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg > /dev/null

    - name: Update Repo
      apt:
        update_cache: true
        force_apt_get: true

    - name: Add mariadb repository
      template:
        src: mariadb.list.j2
        dest: /etc/apt/sources.list.d/mariadb.list

    - name: Install Required Packages
      apt:
        update_cache: yes
        force_apt_get: yes
        name: "{{ required_pkg }}"
        state: latest

    - name: Update the repository
      apt:
        update_cache: yes

    - name: Ensure Nginx service is started and enabled
      service:
        name: nginx
        state: started
        enabled: true

    - name: Ensure MariaDB Server service is started and enabled
      service:
        name: mariadb
        state: started
        enabled: true

    - name: Debug MariaDB status
      command: systemctl status mariadb
      register: mariadb_status

    - name: Print MariaDB status
      debug:
        var: mariadb_status
```

**Run With Sintax Check**

```
ansible-playbook --syntax-check quiz-1_playbook.yml
ansible-playbook quiz-1_playbook.yml
```

**Verification**


```
ls -l ~/quiz-3

ansible webservers -m shell -a "dpkg -l | grep -E 'nginx|mariadb-server|mariadb-client'"

ansible webservers -m shell -a "ls /etc/apt/sources.list.d/nginx.list /etc/apt/sources.list.d/mariadb.list"

ansible webservers -m shell -a "systemctl status nginx | grep 'active (running)'"

ansible webservers -m shell -a "systemctl status mariadb | grep 'active (running)'"

ansible webservers -m shell -a "mysql -V"
```

# THANK YOU





