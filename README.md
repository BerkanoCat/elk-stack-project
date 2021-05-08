## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![image](https://user-images.githubusercontent.com/10915508/117521458-bd266d00-af62-11eb-80a7-cff27cfb0eb9.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. 

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly stable and available, in addition to restricting traffic to the network.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems and system services/metrics.

The configuration details of each machine may be found below.

| Name                 | Function        | IP Address | Operating System |
|----------------------|-----------------|------------|------------------|
| Jump-Box-Provisioner | Gateway         | 10.0.0.8   | Linux            |
| Web-1                | Web Host Server | 10.0.0.5   | Linux            |
| Web-2                | Web Host Server | 10.0.0.6   | Linux            |
| Web-3                | Web Host Server | 10.0.0.9   | Linux            |
| elk-server           | Monitoring      | 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-Box-Provisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
64.52.135.128

Machines within the network can only be accessed by one another, using Docker containers.

A summary of the access policies in place can be found in the table below.

| Name                 | Publicly Accessible | Allowed IP Addresses |
|----------------------|---------------------|----------------------|
| Jump-Box-Provisioner | Yes                 | 64.52.135.128        |
| Web-1                | No                  | 10.0.0.1-254         |
| Web-2                | No                  | 10.0.0.1-254         |
| Web-3                | No                  | 10.0.0.1-254         |
| elk-server           | No                  | 10.0.0.1-254         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because we can rest assured that any of the critical settings of the ELK machine are listed and freely visible in the install-elk.yml file. This reduces confusion and eases troubleshooting.

The playbook implements the following tasks:

- downloads python3-pip and docker
- uses pip to install docker
- ensures that the configured vm has adequate memory to run the necessary container
- pulls and launches the elk container while simultaneously setting up the necessary open port scheme
- ensures that the container is launched every time the virtual machine is booted up


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![image](https://user-images.githubusercontent.com/10915508/117519231-9f073f80-af57-11eb-9559-7e92637aa0d3.png)

the install-elk.yml file is included in this repo, and is reproduced below:

---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: skwee
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: '262144'
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes

### Target Machines & Beats
This ELK server is configured to monitor the three web hosts in the network, at IP addresses 10.0.0.5, 10.0.0.6, and 10.0.0.9

We have installed filebeat and metricbeat onto these machines.

These Beats allow us to collect the following information from each machine:
FILEBEAT: Detects and logs any changes made to the central file system of the machine. Sends this data to Elasticsearch for indexing.

METRICBEAT: Detects and logs any changes made to system metrics. Useful for finding changes in CPU usage, login attempts (warranted or otherwise), RAM statistics, and more. Extremly helpful in detecting malicious behavior.

The Playbook for installing Filebeat is reproduced below, as well as elsewhere in this repo. The playbook for metricbeat is nearly identical and as such will not be reproduced here, but can also be found elsewhere in this repo.

---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/filebeat/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. In our case, we use docker container located on our Jump-Box-Provisioner machine as such.

SSH into the control node and follow the steps below:
- Copy the playbook files to the ansible control node.
- Update the /ansible/hosts file to include the necessary IP addresses of your web servers as well as your elk server.
- Run the playbook, and navigate to http://10.1.0.4*:5601/app/kibana to check that the installation worked as expected0.
-  * replace this IP with that of your elk server, if need be.

