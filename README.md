ansible-tableau-server
=========

Overview
-----------
This repo contains all Ansible roles for installing and configuring a Tableau Server cluster on Amazon Linux 2.

Requirements
------------
The machine must meet the [Minimum Hardware Requirements](https://help.tableau.com/current/server-linux/en-us/server_hardware_min.htm)
 for Tableau Server, which is currently:

- Processor: 64-bit
- RAM: 16GB
- CPU: 4-core
- Free Disk Space: 15GB

The role has only been tested on the following software:

- [Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/)

Playbooks
--------------
### [`tableau-server-install`](roles/tableau-server-install/tasks/install_tableau.yml)
This role installs Tableau Server 2019.04.1+ and drivers on Amazon Linux 2.

##### Required Vars
- **`tableau_server_version`**
    - Version of Tableau Server to be installed. 
- **`drivers`**
    - List of drivers to be installed (RPM package urls).
- **`tsm_user` and `tsm_password`**
    - Username and password for TSM. 
    - TSM is used to manage installation and configuration of Tableau Server.

##### Using Role Example
[tableu-server-install.yml](tableau-server-install.yml)

```yaml
    - hosts: all
      become: yes
      roles:
        - tableau-server-install
```

##### Run Example
    ansible-playbook -i 10.0.10.100, -u ec2-user --key-file=tableau-server.pem -e '{"tableau_server_version":"2019.4.1", "tsm_user":"tsmadmin", "tsm_password":"coxinha123", "drivers":["https://downloads.tableau.com/drivers/linux/yum/tableau-driver/tableau-postgresql-odbc-09.06.0500-1.x86_64.rpm"]}' tableau-server-install.yml

### [`tableau-server-primary-setup`](roles/tableau-server-primary-setup/tasks/setup.yml)
This role configures Tableau Server previously installed as the primary machine in the cluster.

##### Dependencies
- A successful execution of the role table-server-install, as demonstrated in the previous example.
- Create file registration.json in roles/tableau-server-primary-setup/files. Check registration.json.example in project.
- Create file settings.json in roles/tableau-server-primary-setup/files. Check settings.json.example in project.

##### Required Vars
- **`tsm_user`**
    - Username for TSM.
    - TSM is used to manage installation and configuration of Tableau Server.
- **`licenses`**
    - List of licenses provided by Tableau. 
- **`tableau_admin_user` and `tableau_admin_password`**
    - Username and password for Tableau Server.
    - As a administrator on Tableau Server, you can access admin settings to configure users, projects, and to do other content-related tasks. 

##### Using Role Example
[tableu-server-primary-setup.yml](tableau-server-primary-setup.yml)

```yaml
    - hosts: all
      become: yes
      roles:
        - tableau-server-primary-setup
```

##### Run Example
    ansible-playbook -i 10.0.10.100, -u ec2-user --key-file=tableau-server.pem -e '{"tsm_user":"tsmadmin", "tableau_admin_user": "admin", "tableau_admin_password":"coxinha123", "licenses": ["XXXX-XXXX-XXXX-XXXX-XXXX"]}' tableau-server-primary-setup.yml

### [`tableau-server-worker-setup`](roles/tableau-server-worker-setup/tasks/setup.yml)
This role configures Tableau Server previously installed as the worker machine in the cluster.

##### Dependencies
- A successful execution of the role table-server-install, as demonstrated in the previous example.
- Primary machine running TSM.
- [Download bootstrap.json file with credentials](https://help.tableau.com/current/server-linux/en-us/install_additional_nodes.htm) and copy to roles/tableau-server-worker-setup/files
- Worker machine needs access primary machine ports:
    - 22
    - 8000 - 9000
    - 27000 - 27009

##### Required Vars
- **`tsm_user`**
    - Username for TSM.
    - TSM is used to manage installation and configuration of Tableau Server.

##### Using Role Example
[tableu-server-worker-setup.yml](tableau-server-worker-setup.yml)

```yaml
    - hosts: all
      become: yes
      roles:
        - tableau-server-worker-setup
```

##### Run Example
    ansible-playbook -i 10.0.10.100, -u ec2-user --key-file=tableau-server.pem -e '{"tsm_user":"tsmadmin"}' tableau-server-worker-setup.yml
    
License
-------
MIT

Author Information
-------

This roles was created in 2020 by Alex Pereira.