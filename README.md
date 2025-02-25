# Automating Node.js Application Deployment

## Project Overview

This project demonstrates how I automated the deployment of a Node.js application using Ansible on a DigitalOcean droplet. The automation handles, dependency installation, user creation, and application deployment.

## Technologies Used

- **Ansible**: For automation and configuration management
- **Node.js**: Runtime for the web application
- **DigitalOcean**: Cloud hosting provider
- **Linux**: Operating system for the server

## Implementation Steps

### 1. Server Creation

I created a new Ubuntu server on DigitalOcean and configured SSH access for secure remote management.

![droplet](https://github.com/Princeton45/nodejs-ansible-deploy/blob/main/images/droplet.png)

### 2. Ansible Configuration

I wrote an inventory file to define the server details and connection parameters for the Ansible controller node to connect to the new droplet server.

`159.223.167.141 ansible_ssh_private_key_file=~/id_ed25519 ansible_user=root`


### 3. Playbook Development

I created an Ansible playbook that:
- Installs Node.js and npm dependencies
- Creates a dedicated Linux user for running the application
- Copies a Node.js tar file (which contains the application), unpacks the file and then starts the application
- Verifies that the App is successfully running.

```yaml
---
- name: Install node and npm
  hosts: 159.223.167.141
  tasks:
    - name: Update apt repo and cache
      apt:
        update_cache: yes
        force_apt_get: yes
        cache_valid_time: 3600
    - name: Install nodejs and npm
      apt:
        pkg:
          - nodejs
          - npm

- name: Create new linux user for node app
  hosts: 159.223.167.141
  tasks:
    - name: Create linux user
      user:
        name: nodejs
        comment: Nodejs service account
        group: admin


- name: Deploy nodejs app
  hosts: 159.223.167.141
  become: True
  become_user: nodejs
  tasks:
    - name: Copy nodejs folder to a server
      copy: 
        src: /home/princeton/nodejs-app-main.tar.gz
        dest: /home/nodejs/app-1.0.0.tgz
    - name: Unpack the nodejs tar file
      unarchive: 
        src: /home/princeton/nodejs-app-main.tar.gz
        dest: /home/nodejs
    - name: Install dependencies
      npm:
        path: /home/nodejs/nodejs-app-main/
    - name: Start the application
      command:
        chdir: /home/nodejs/nodejs-app-main/app/
        cmd: node server
      async: 1000
      poll: 0
    - name: Ensure app is running
      shell: ps aux | grep node
      register: app_status
    - debug:
        msg: "{{app_status.stdout_lines}}"
```


### 6. Execution Results

After running the playbook, the application was successfully deployed and running

![ps-aux](https://github.com/Princeton45/nodejs-ansible-deploy/blob/main/images/ps-aux.png)

![ansible-playbook](https://github.com/Princeton45/nodejs-ansible-deploy/blob/main/images/ansible-playbook.png)

