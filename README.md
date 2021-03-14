# Mikhail Erick

## Project One: ELK Stack

The files posted to this GitHub are commands and methods used to create the ELK Stack project.

The Files included inside this project can be used to recreate the entire project or specific parts. .YML files can be used for select portions, while files inside of the Linux folder include homework questions and answers.

This GitHub repositary contains the following work:
- Layout of repositary
- Connections
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### GitHub repositary layout

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
Load balancers help ensure environment availability through distribution of incoming data to web servers. Jump boxes allow for more easy administration of multiple systems 
and provide an additional layer between the outside and internal assets.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the event logs and system metrics.
- Filbeats watch for log directories or specific log files. 
- Metricbeat helps you monitor your servers by collecting metrics from the system and services running on the server.

The configuration details of each machine may be found below.


| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| JumpBoxProvisioner | Gateway  | 10.0.0.4   | Linux (Ubuntu)            |
| Web 1   | Server         | 10.0.05           | Linux (Ubuntu)              |
| Web 2   | Server         | 10.0.06          | Linux (Ubuntu)               |
| ELK Server | Log Server   | 10.1.0.4           | Linux (Ubuntu)  |

### Connections

Machines connected to the Virtual Network are not accessable from the internet until setup allows.

The JumpBoxProvisioner is the only machine allowed to be connected to via my personal IP. This is done through SSH to reduce posibilities of brute forcing passwords.

Machines within the network can only be accessed by the Jump Box.


| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes            | Personal    |
| Web 1         | No                    |  10.0.0.5                   |
| Web 2         | No                    |  10.0.0.6                    |
| ELK Server  |    Yes     |   Personal    |
(List of the machines and accessibility)

### Ansible/Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because servcies running can be limited, system installation and update can be streamlined, and processes become more replicable.
I used ansible to configure all of the Docker containers and scripts. This allowed me to quickly create, install and setup the machines (Web1&2) and the ELK Server.

The playbook completes the following tasks:
- installs docker.io, docker module and the pip3 application
```bash
  # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

  # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

  # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
```   
- increases the virtual memory (for the virtual machine we will use to run the ELK server)
```bash
  # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144
```
- sysctl module
```bash
  # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
```
- downloads and launches Docker for ELK
```bash
# Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

### Target Machines & Beats
This ELK server is configured to monitor:
- Web 1 (10.0.0.5)
- Web 2 (10.0.0.6)

We have installed - filebeats on the server

These Beats allow us to collect the following information from each machine:
- Filebeat is a log data shipper for local files. Installed as an agent on your servers, Filebeat monitors the log directories or specific log files, tails the files, and forwards them either to Elasticsearch or Logstash for indexing. An examle of such are the logs produced from the MySQL database supporting our application. 


### Using the Playbook
In order to use the playbook, you will need to have an Ansible machine running.

SSH into the JumpBoxProvisioner (Ansible control) and follow the steps below:
- Copy the configuration file from your Ansible container to your Web VM's 
- Update the /etc/ansible/hosts file to include the IP address of the Elk Server VM and webservers.
- Run the playbook, and navigate to http://[Elk_VM_Public_IP]:5601/app/kibana to check that the installation worked as expected.

- _Which file is the playbook?_ 
Docker Containers

- _Where do you copy it?_
cp /etc/ansible/files/filebeat-config.yml > /etc/filebeat/filebeat.yml

- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
update filebeat-config.yml -- specify which machine to install by updating the host files with ip addresses of web/elk servers and the apache2 webapp.

- _Which URL do you navigate to in order to check that the ELK server is running?
http://[your.ELK-VM.External.IP]:5601/app/kibana.

#### Installing Filebeat
---
    - name: Installing and Launch Filebeat
      hosts: webservers
      become: yes
      tasks:
       # Use command module
    - name: Download filebeat .deb file
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
       # Use command module
    - name: Install filebeat .deb
      command: dpkg -i filebeat-7.4.0-amd64.deb
       # Use copy module
    - name: Drop in filebeat.yml
      copy:
       src: /etc/ansible/files/filebeat-config.yml
         dest: /etc/filebeat/filebeat.yml
       # Use command module
     - name: Enable and Configure System Module
       command: filebeat modules enable system
      # Use command module
    - name: Setup filebeat
      command: filebeat setup
      # Use command module
    - name: Start filebeat service
      command: service filebeat start
      # Use systemd module
    - name: Enable service filebeat on boot
      systemd:
         name: filebeat
         enabled: yes
