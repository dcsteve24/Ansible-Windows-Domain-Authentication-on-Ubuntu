SSSD_AUTH
=========

This role enables SSSD-Authentication to a windows domain, allowing users to login with just username; not username@domain.

Upon first login, the user's home directory will be made

Note: this will overwrite any existing configurations in the following:
- /etc/krb5.conf
- /etc/ntp.conf 
- /etc/realmd.conf
- /etc/sssd/sssd.conf

backups are taken, but you should be aware of this prior to running.

Requirements
------------

- ability to ping and DNS resolve the domain/domain controller(s)
- an account capable of adding clients to the domain

Role Variables
--------------

| Variable  | Required | Default | Description
| ------------- | ------------- | ------------- | -------------|
| realm_domain  | Yes  | N/A | used to hold your domain |
| krb5_default_realm  | Yes  | N/A | where the kerberos authentication occurs (typically same as realm_domain). Must be in all CAPS. |
| realm_ad_ou | Yes | N/A | the OU or CN (in LDAP form) to place the PC when joined to the domain |


Example Playbook
----------------

    - hosts: all
      roles:
         - sssd_auth

Author Information
------------------

Steven Craig, 31Jan19
