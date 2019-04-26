# Ansible-Windows-Domain-Authentication-on-Linux

SSSD_AUTH
=========

This is an ansible play that will enable windows active directory domain authentication on Linux machines. This play is based on the guide made here: http://ricktbaker.com/2017/11/08/ubuntu-16-with-active-directory-connectivity/

This has been tested on Ubuntu 16 and Ubuntu 18.

# Cons:
while the guide states that adding lines to the sssd.conf like
 - simple_allow_users = user1,user2
 - simple_deny_users = user1,user2
 - simple_allow_groups = group1,group2
 - simple_deny_groups = group1,group2

will control accesses to the machine; this did not work for our environment. Once I tied the domain authentication into the devices, anybody that had a domain account could log in. However, only those I explicitly allowed to sudo through the specified domain group, could do so. I managed to control access by setting a AllowedGroup setting on the ssh configs; physical access to the machines were not possible because of virtualization. But be aware of this; if a fix is found, I'd be curious and edit this as needed.

# Warning:
running this play will edit the following files:
  - /etc/ntp.conf
  - /etc/realmd.conf
  - /etc/krb5.conf
  - /etc/pam.d/common-session
  - /etc/sssd/sssd.conf
  - /etc/nsswitch.conf
  - /etc/sudoers

If your target is a fresh attempt at this, this will not affect anything. If you have prior configs with these files, use at your own risk. Backups will be taken in the event this does mess up your environment. They will be stored at the location of the file as <file>.timestamp.

Requirements
------------
**Prereqs**
- ability to ping and DNS resolve the domain/domain controller(s)
- an account capable of adding clients to the domain

1. Edit ./roles/playbooks/sssd_auth.yml and fill in your environment info
2. Edit ./roles/sssd_auth/vars/main.yml and fill in the variables that require your input. Also edit any defaults as desired.
2. Edit ./roles/sssd_auth/vars/secrets.yml and fill in the variables that require your input.
3. Encrypt the secrets file with ansible-vault encrypt ./roles/sssd_auth/vars/secrets.yml
4. When all the information has been changed for your environment, run the play. I typically run this play with ansible-playbook -kK --ask-vault-pass path_to_sssd_auth.yml

Role Variables
--------------

| Variable  | Location | Required | Default | Description
| ------------- | ------------- | ------------- | ------------- | ------------- |
| krb5_conf | ./roles/sssd_auth/defaults/main.yml | Yes | /etc/krb5.conf | Path to the krb5 configuration |
| krb5_default_realm  | ./roles/sssd_auth/vars/main.yml | Yes  | N/A | where the kerberos authentication occurs (typically same as realm_domain). Must be in all CAPS. |
| kerberos_user | ./roles/sssd_auth/vars/secrets.yml | Yes | N/A | The user that can add computers to the domain |
| kerberos_user_password | ./roles/sssd_auth/vars/secrets.yml | Yes | N/A | The password of the user that can add computers to the domain |
| nsswitch_conf | ./roles/sssd_auth/defaults/main.yml | Yes | /etc/nsswitch.conf | Path to the nsswitch configuration |
| ntp_conf | ./roles/sssd_auth/defaults/main.yml | Yes | /etc/ntp.conf | Path to the ntp configuration |
| pam_common | ./roles/sssd_auth/defaults/main.yml | Yes | /etc/pam.d/common-session | Path to the pam common-session configuration |
| realm_ad_ou | ./roles/sssd_auth/vars/main.yml |Yes | N/A | the OU or CN (in LDAP form) to place the PC when joined to the domain |
| realmd_conf | ./roles/sssd_auth/defaults/main.yml | Yes | /etc/realmd.conf | Path to the realmd configuration |
| realm_domain  | ./roles/sssd_auth/vars/main.yml | Yes  | N/A | used to hold your domain |
| sssd_conf | ./roles/sssd_auth/defaults/main.yml | Yes | /etc/sssd/sssd.conf | Path to the sssd configuration |
| sudo_group | ./roles/sssd_auth/vars/main.yml |Yes | N/A | Adds the specified group to allow the ability to sudo|
| sudo_config | ./roles/sssd_auth/defaults/main.yml | Yes | /etc/sudoers | Path to the sudoers configuration |

Example Playbook
----------------

    - hosts: all
      become: true
      roles:
         - sssd_auth

Author Information
------------------

Steven Craig, 31Jan19
Edited By Steven Craig 24Apr19
