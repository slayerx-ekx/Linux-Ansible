# This is if you don't like text you can watch this image

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
- name: Use Variables in Playbook
  hosts: pod-username-managed2
  remote_user: student
  become: true
  vars:
    required_Pkg:
      - apache2
      - python3-urllib3
    web_Service: apache2
    content_File: "adinusa lab quiz variables - username"
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
        url: http://pod-username-managed2/index.html
        return_content: no
        status_code: 200
```
