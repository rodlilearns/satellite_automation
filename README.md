# satellite_automation
An example of automating Red Hat Satellite installation and configuration.

## Purpose

Red Hat Satellite is an infrastructure management product which is great for managing large fleets of Red Hat Enterprise Linux machines.  
It can manage subscriptions, act as the source of truth for verified content, manage life-cycles, keep systems secure and compliant, as well as provision systems.  

This project uses Ansible (a command-line I.T. automation tool written in Python) to install and configure a Red Hat Satellite server.  
This project was written with idempotency and consistency in mind, meaning it can be run as many times as desired, without making changes that have already been made.  
The Ansible code is written in yaml. It is declarative, and therefore easier to understand when debugging, and can be shared with other teams for ease when communicating details is required.  

## Requirements

### Collection Source Configuration  
Configure Collection sources.  
`$ vim ansible.cfg`  

```
[galaxy]
server_list = automation_hub, galaxy

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=<INSERT_TOKEN>
```
Note: Make sure to check `console.redhat.com` for the exact Red Hat Automation Hub `url` and `auth_url` inputs, they're most likely different.  

### Ansible Collections
The required Ansible collections are in `collections/requirements.yml`.  

Install required collections  
`$ ansible-galaxy collection install -r collection/requirements.yml`  

### Execution Environment Image
Log into Red Hat container registry.
`$ podman login registry.redhat.io`  
Note: Make sure that you have set up two factor authentication with your Red Hat registry account. Otherwise, you will not be able to log in successfully.  

Download the default container image.
`$ ansible-navigator images --eei registry.redhat.io/ansible-automation-platform-22/ee-supported-rhel8:1.0.0-233`  

### Credentials
The following credentials need to be set up so that the playbook can access them within the execution environment.  

#### Ansible Become Password
Note: The leading space here is necessary to keep the command out of the command history.
`$  export ansible_become_password=<ENTER_ANSIBLE_BECOME_PASSWORD>`

#### Ansible Vault Password
```
$ touch ~/.vault_password
$ chmod 600 ~/.vault_password
# The leading space here is necessary to keep the command out of the command history
$  echo my_password >> ~/.vault_password
# Link the password file into the current working directory
$ ln ~/.vault_password .
# Set the environment variable to the location of the file
$ ANSIBLE_VAULT_PASSWORD_FILE=.vault_password
$ ansible-navigator run site.yml
```
https://ansible-navigator.readthedocs.io/en/latest/faq/#how-can-i-use-a-vault-password-with-ansible-navigator

## Implementation
This project uses the `satellite_install.yml` Ansible playbook to install a Red Hat Satellite server.

Then, the `satellite_configure.yml` Ansible playbook configures the existing Red Hat Satellite.  
The items this playbook configures include:
1. Organisations.
2. Locations.
3. Enable RPMs (RHEL 8, RHEL 9, Satellite Client 6 for RHEL 9).
4. Syncs Red Hat repositories.
5. Life-cycle Environments (Dev, Test, Prod).
6. Content Views (RHEL9_Base_CV, )

## How to use
Provision a host for Satellite server.  

Set up public key authentication.  
`$ ssh-copy-id ~/.ssh/id_rsa.pub <INSERT_USER>@<INSERT_HOST>`

Clone the satellite_automation project.  
`$ git clone https://github.com/rodlilearns/satellite_automation.git`  

Create an Ansible Vault file containing your secrets.  

`$ ansible-vault create host_vars/vault.yml`  

```
---
vault_satellite_server_url: <INSERT_SAT_SERVER_URL>
vault_satellite_username: <INSERT_SAT_USERNAME>
vault_satellite_password: <INSERT_SAT_PASSWORD>
vault_rh_username: <INSERT_RH_USERNAME>
vault_rh_password: <INSERT_RH_PASSWORD>
```
`:wq`  

Run the playbook to install Satellite server.  
`$ ansible-playbook satellite_install.yml -b --become-method sudo --become-user root -u <INSERT_USER> -kK --ask-vault-pass -vvv`  
or  
`$ ansible-navigator run satellite_install.yml -m stdout --eei ee-supported-rhel8:1.0.0-233 --pp missing -penv ansible_become_password`

Run the playbook to configure Satellite server.  
`$ ansible-playbook satellite_configure.yml -b --become-method sudo --become-user root -u <INSERT_USER> -K --ask-vault-pass -vvv`
or  
`$ ansible-navigator run satellite_configure.yml -m stdout --eei ee-supported-rhel8:1.0.0-233 --pp missing -penv ansible_become_password`

## Recommended reading
* [Red Hat Satellite 6.12: Installing Satellite Server in a Connected Network Environment](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.12/html/installing_satellite_server_in_a_connected_network_environment/index)  
* [Ansible Content: redhat.satellite_operations](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/satellite_operations/)
* [Ansible Content: redhat.satellite](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/satellite/)
* [GitHub Repository: redhat.satellite_operations](https://github.com/RedHatSatellite/satellite-operations-collection)
* [GitHub Repository: redhat.satellite](https://github.com/RedHatSatellite/satellite-ansible-collection)
* [Using Ansible Vault with ansible-navigator](https://ansible-navigator.readthedocs.io/en/latest/faq/#how-can-i-use-a-vault-password-with-ansible-navigator)  