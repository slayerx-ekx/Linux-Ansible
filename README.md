# Hallo World!

## Thank you

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
pod-username-managed2
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

pastikan di luar folder saat check atau verifikasi

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


