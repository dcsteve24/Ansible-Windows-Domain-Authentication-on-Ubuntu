# Ansible-Windows-Domain-Authentication-on-Linux

This is an ansible play that will enable windows active directory domain authentication on Linux machines. This play is based on the guide made here: http://ricktbaker.com/2017/11/08/ubuntu-16-with-active-directory-connectivity/

This has been tested on Ubuntu 16 and Ubuntu 18.

# Cons:
while the guide states that adding lines to the sssd.conf like
 - simple_allow_users = user1,user2
 - simple_deny_users = user1,user2
 - simple_allow_groups = group1,group2
 - simple_deny_groups = group1,group2

will control accesses to the machine; this did not work for our enviornment. Once I tied the domain authentication into the devices, anybody that had a domain account could log in. However, only those I explicitly allowed to sudo through the specified domain group, could do so. I managed to control access by setting a AllowedGroup setting on the ssh configs; physical access to the machines were not possible because of virtualization. But be aware of this; if a fix is found, I'd be curious and edit this as needed.

# Warning:
running this play will edit the following files:
- /etc/krb5.conf
- /etc/ntp.conf
- /etc/realmd.conf
- /etc/sssd/sssd.conf

If your target is a fresh attempt at this, this will not affect anything. If you have prior configs with these files, use at your own risk. Backups will be taken in the event this does mess up your enviornment. They will be stored at the location of the file as <file>.timestamp.

# How To Use:
This play is expected to be placed in your root ansible folder; default of /etc/ansible/

When downloaded, the following edits will need to be made:
1. Edit ./playbooks/sssd_auth.yml - Add the user you SSH/sudo with and change the hosts as needed for your enviornment.
2. Edit ./roles/sssd_auth/templates/krb5.conf.j2 - Change default_realm to your domain in all caps.
3. Edit ./roles/sssd_auth/templates/ntp.conf.j2 - point the server line (line 21) to your domain controller. Add another line stating the same thing but pointing to alternate domain controllers if you have multiple.
4. Edit ./roles/sssd_auth/templates/realmd.conf.j2 - change [your_domain] to your domain
5. Edit ./roles/sssd_auth/templates/sssd.conf.j2 - change the following:
   - domains = your_domain
   - [domain/your_domain]
   - ad_domain = your_domain
   - krb5_realm = CAPITAL_YOUR_DOMAIN
6. Edit ./roles/sssd_auth/vars/main.yml - enter your specific enviornment information; note the container should be in LDAP format, i.e. cn=computers,domain=your,domain=domain,domain=here. If you don't know it, you can obtain this info from windows Users and Computers; enable Advanced features under the view tab -->  right click desired destination --> Properties --> Attribute Editor tab --> find DistinguishedName. That entry is what you need to enter.
7. Edite ./roles/sssd_auth/vars/secrets.yml - enter your username and password of the user that can add or remove clients in the domain. Encrypt this file afterwards with ansible-vault encrypt path_to_file

When all the information has been changed for your enviornment, run the play. I typically run this play with ansible-playbook -kK --ask-vault-pass path_to_sssd_auth.yml
