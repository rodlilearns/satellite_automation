# satellite_automation
A template for automating Red Hat Satellite installation and configuration.

## Purpose

Red Hat Satellite is an infrastructure management product which is great for managing large fleets of Red Hat Enterprise Linux machines.  
It can manage subscriptions, act as the source of truth for verified content, manage life-cycles, keep systems secure and compliant, as well as provision systems.  

This project uses Ansible (a command-line I.T. automation tool written in Python) to install and configure a Red Hat Satellite server.  
This project was written with idempotency and consistency in mind, meaning it can be run as many times as desired, without making changes that have already been made.  
The Ansible code is written in yaml. It is declarative, and therefore easier to understand when debugging, and can be shared with other teams for ease when communicating details is required.  

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

Install required collections  
`$ ansible-galaxy collection install -r collection/requirements.yml`  

Run the playbook to install Satellite server.  
`$ ansible-playbook satellite_install.yml -b --become-method sudo --become-user root -u <INSERT_USER> -kK --ask-vault-pass -vvv`  

Run the playbook to configure Satellite server.  
`$ ansible-playbook satellite_configure.yml -b --become-method sudo --become-user root -u <INSERT_USER> -K --ask-vault-pass -vvv`