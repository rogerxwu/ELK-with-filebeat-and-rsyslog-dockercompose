# ELK-with-filebeat-and-rsyslog-dockercompose
A containerized log analysis service, built with elastic + Logstash + Kibana + filebeat + rsyslog.

<h2>Project background:</h2>
A large number of network devices and servers are usually deployed in an enterprise environment. When a device fails, decentralized log management will make it difficult to recover the log of the failed device. The troubleshooting process requires cumbersome collection of log information of the associated device. And it is difficult to monitor the abnormalities of the equipment in real time. The centralized log management analysis service can effectively improve the efficiency of troubleshooting, and can detect man-made or non-man-made equipment abnormalities in time.

Reference: 
This project refers to the source code of https://github.com/gnokoheat/elk-with-filebeat-by-docker-compose, and carries out follow-up development

<h2>Project Workflow:</h2>
device logs -> server rsyslog -> filebeat -> logstash -> elasticsearch -> kibana

<h2>Project tutorial:</h2>
<h3>Configure the log server and clients to achieve centralized log reception</h3>
<h4>1. Configure the log server: replace the rsyslog.conf file to /etc/rsyslog.conf</h4>

```
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$template Remote, "var/log/remote_log/%$YEAR%-%$MONTH%-%$DAY%/%fromhost-ip%.log"
:fromhost-ip, !isequal, "127.0.0.1" ?Remote
```
<h4>2. Configure clients such as network devices</h4>
Use Ansible to automate logging service on client


```
---
- name: us fontana all sw enable logging
  hosts: Fontana_switch
  gather_facts: no
  connection: network_cli
  tasks:
    - name: enable logging service
      cisco.ios.ios_config:
        commands:
          - logging host 172.30.249.149
          - logging trap 7
          - logging source-interface vlan 70
        save_when: modified
```


<h4/>3. Check the log collection

```
cd /var/log/remote_log
```


<h3>Run ELKB service</h3>
<h4/>1. Install docker and docker-compose on the server
<h4/>2. Download the code
<h4/>3. Modify the docker-compose.yml and filebeat.yml files according to the log storage location, ensure that the filebeat in docker-compose is mounted to the correct log folder, and confirm filebeat that the file is correctly used as the input source

```
# filebeat.yml
filebeat.inputs:
- type: log   # depends on the log file you collected from the devices
  enabled: true
  paths:
    - /usr/share/filebeat/remote_log/2021*/*.log    #suggest to create a remote_log dir to store all remotely collected logs
    
# docker-compose.yml
volumes:
      - /var/run/docker.sock:/host_docker/docker.sock
      - /var/lib/docker:/host_docker/var/lib/docker
      - ../../../var/log/remote_log:/usr/share/filebeat/remote_log  #use your log file location to update this line
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml

```
<h4/>4. Run docker-compose</h3>

```
docker-compose up -d
```
<h4/>5. Modify the logstash configuration file and template file as required
