# Ansible Pre-Requisite Check

This repository contains an Ansible playbook that checks the following roles..

## Roles and Validation

The following roles will be checked and validated by this script:
- **network_connectivity_check**
- **os_services_check**
- **packages_check**
- **ports_check_role**

This script is the property of **BBI** and is created and maintained by the **BBI DevOps team**.

## Prerequisites

Ensure that you have the following before running the playbook:

- A Linux-based system (tested on CentOS/RHEL)
- **Ansible installed**
- **An inventory file** with the target hosts
- **Encrypted ports file (`ports.json`)**
- **Correct permissions on the output directory**
- **Ansible logging enabled**
- **Database hostname, username, and password added to `roles/ports_check_role/vars/network_secrets.yml`**
- **Active Directory hostname and port added to `roles/network_connectivity_check/vars/main.yml`**
- **NTP server hostname and port added to `roles/network_connectivity_check/vars/main.yml`**

## Installation and Setup

### Install Ansible

Run the following command to install Ansible:

```bash
yum install ansible -y
```

### Configure the Inventory File

Edit the `inventory/inventory.yml` file and add the hostnames of the machines where the playbook will run.

Example:

```yaml
all:
  hosts:
    master1:
      ansible_host: master1.example.com
    worker1:
      ansible_host: worker1.example.com
    cm_host:
      ansible_host: cm_host.example.com
```

### Configure Network Secrets

Edit the `roles/ports_check_role/vars/network_secrets.yml` file to add the required credentials and hostnames:

```yaml
ad_server: "192.168.1.1"
ad_port: 389
ad_bind_user: "CN=Admin,CN=Users,DC=example,DC=com"
ad_bind_password: "SuperSecretPassword"

db_server: "192.168.1.2"
db_port: 1521
db_user: "dbadmin"
db_password: "SuperSecretDBPassword"
db_service: "ORCL"

ntp_server: "192.168.1.3"
ntp_port: 123
```

### Encrypt the Network Secrets File

After editing the `network_secrets.yml` file, encrypt it with Ansible Vault:

```bash
ansible-vault encrypt roles/ports_check_role/vars/network_secrets.yml
```

To edit this file later, use:

```bash
ansible-vault edit roles/ports_check_role/vars/network_secrets.yml
```

### Encrypt the Ports File

The playbook reads from a JSON file containing port information. Encrypt it with **Ansible Vault** using the password "changeit".

```bash
ansible-vault encrypt roles/ports_check_role/vars/ports.json
```

When prompted, enter the password:

```
changeit
```

To edit the file later, use:

```bash
ansible-vault edit roles/ports_check_role/vars/ports.json
```

### Create the Output Directory

The playbook will store results in `/tmp/ansible_pre_req_check/`. Ensure it exists and has correct permissions:

```bash
mkdir -p /tmp/ansible_pre_req_check
chmod 777 /tmp/ansible_pre_req_check
```

### Enable Ansible Logging

Set up logging for Ansible:

```bash
echo 'export ANSIBLE_LOG_PATH="/tmp/ansible_pre_req_check/ansible_pre_req_check_$(date +%Y-%m-%d_%H-%M-%S).log"' >> ~/.bashrc
source ~/.bashrc
```

## Running the Playbook

Run the playbook using:

```bash
ansible-playbook -i inventory/inventory.yml playbook.yml --ask-vault-pass -f 50
```

When prompted for a password, enter:

```
changeit
```

## Output Location

The playbook will execute on all hosts and generate the output in:

```
/tmp/ansible_pre_req_check/ports_check.csv
```

To view the results:

```bash
cat /tmp/ansible_pre_req_check/ports_check.csv
```

## Example Output

```csv
source_hostname,destination_hostname,port,connection_status,service,daemon,comment,test_type
master1.example.com,worker1.example.com,25,ACCESSIBLE,Email Notification,SMTP ,SMTP ( Default - Non SSL),Port_Connectivity_Test
worker1.example.com,cm_host.example.com,2049,BLOCKED,HDFS,NFS gateway,nfs port (nfs3.server.port) ,Port_Connectivity_Test
...
```

## Troubleshooting

### Issue: Getting "Permission Denied" on Output File

Run:

```bash
chmod 777 /tmp/ansible_pre_req_check/ports_check.csv
```

### Issue: Playbook Fails Due to Missing Password

If you forgot to encrypt the `ports.json` file, run:

```bash
ansible-vault encrypt roles/ports_check_role/vars/ports.json
```

## Summary

- Install Ansible
- Edit the inventory file
- Edit and encrypt `network_secrets.yml`
- Encrypt `ports.json`
- Add required hostnames and ports to `roles/network_connectivity_check/vars/main.yml`
- Create `/tmp/ansible_pre_req_check/`
- Enable logging
- Run the playbook

## License

This project is for internal use. Modify as needed.

## Contributions

Feel free to improve this repository! Open a pull request for enhancements.

Maintainer: *BBI DevOps Team*
Company: *BBI*


