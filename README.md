# satellite_automation
A template for automating Red Hat Satellite configurations

Some generic configuration tasks for Red Hat Satellite include:  

- Implement Role Based Access Control for the Satellite.  
- Configure digital certificates.  
- Configure organizations.  
- Configure locations.  
- Configure and synchronize content.  
- Create sync plan.  
- Configure Life-cycle environments.  
- Implement SCAP content that aligns with a security profile.  

This project uses the `satellite_configure.yml` Ansible playbook to configure an existing Red Hat Satellite.  
The items this playbook configures include:
1. Organisations.
2. Locations.
3. Enable RPMs (RHEL 8, RHEL 9, Satellite Client 6 for RHEL 9).
4. Syncs Red Hat repositories.
5. Lifecycle Environments (Dev, Test, Prod).
6. Content Views (RHEL9_Base_CV, )