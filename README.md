Ansible Prep
=========

This role is used for first-time deployment of an Ansible user, and lockdown/deletion of the stock system account

WARNING: Read the requirements section as this role deletes a user account

Preface
-------

This role has evolved from an original requirement to deploy / manage multiple Raspberry Pi systems. Raspbian by default creates a user called 'pi' with the password 'raspberry'. In the original model of this role we used this account to deploy a new privilaged user and lock down the pi user. This was done on the first run of the role using the pi user.
A second run of the role (this time as the ansible user that we just created) would subsequently destroy the pi account.

Going forward, I have expanded the role to support most common Linux OSes.



Requirements
------------

Using the raspberry pi model we assume the following:
- An account exists on the system with the same name as the os (this uses the default_usr var). It assumes that if you use CentOS then your default user account will be centos.
This account must have sudo privilages, or if you have a root account on the system, it must have the ability to su to root. For auditing and security reasons it is better practice to not have a root account any more, and to use privilage escalation
WARNING: This account will be deleted! Only use it for the initial setup

- SSH access enabled. On a raspberry pi create a blank file called 'ssh' on the boot partition of the SD Card to enable this at first boot (useful in Headless mode)

You will need:
- Ansible account username
- Ansible account password - Create this using: $ python -c 'import crypt; print crypt.crypt("This is my Password", "$1$SomeSalt$")'
- Ansible account SSH public key - Create this using: $ ssh-keygen   then copy in your public key to this field. alternatively specify a file name and store it under files, or store it online and provide the URL. See https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html

These should be set to variables in your playbook. See the Role Variables and Example Playbook sections to see examples of these.

Important Note: Once you've run your python commmand to create an encrypted password, it may be a good idea to exist from the shell used to create it, then edit your ~/.bash_history file to delete the text record of this password.

Role Variables
--------------

Playbook Variables
- ansible_usr: Ansible account username
- ansible_pwd: Desired password, this must be encrypted! - use the following command to generate this: $ python -c 'import crypt; print crypt.crypt("This is my Password", "$1$SomeSalt$")'
- ansible_key: The public half of the SSH keypair generated: use the following command to generate this: $ ssh-keygen
- default_usr: This is set across various OS-specific var files and uses the os name (lowercase). It is not setable yet.

Role Variables
- sudo_grp: The default sudo groups that a privilaged account needs to be added to for Sudo privilages. These are set at the OS family level.
 - Family/Redhat: wheel
 - Family/Debian: sudo, adm
- package_install: The list of packages we want to install on a per-family or per-OS basis. NOTE: RedHat OS Family has been overridden at the OS specific level as CentOS/RHEL 7 and 8 have slightly different requirements
- default_usr: This is the default user created on each OS - this is needed for the first run of the ansible-prep role but after the first run and the Ansible user has been created, this account is obsolete and can be deleted. This variable specifies which account to look for on each OS and lockdown or delete.
 - OS/RedHat: redhat
 - OS/CentOS: centos
 - OS/Ubuntu: ubuntu
 - OS/Debian: debian
 - OS/Raspbian: pi


Dependencies
------------

No dependencies

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      become: yes
      gather_facts: true
      roles:
        - role: '~/ansible/roles/ansible-prep'
      vars:
        ansible_usr: ansible
        ansible_pwd: $1$SomeSalt$UqddPX3r4kH3UL5jq5/ZI.
        ansible_key: ssh-rsa enter_rsa_publickey_here user@system


To run the playbook (uses sudo):
First run - Raspberry Pi: (password: raspberry):
ansible-playbook -i ~/ansible/server/inventory/hosts ~/ansible/server/playbooks/ansible-prep.yml -u pi -k

First run - Debian (uses su):
ansible-playbook -i ~/ansible/server/inventory/dev ~/ansible/server/playbooks/ansible-prep.yml --become-method su -u <user> -k -K

First run - CentOS (uses sudo):
ansible-playbook -i ~/ansible/server/inventory/dev ~/ansible/server/playbooks/ansible-prep.yml -u <user> -k

Second Run: (ansible auth via SSH keypair. SUDO via preset password)
$ ansible-playbook --private-key ansible/server/ssh/id_rsa -i ~/ansible/server/inventory/hosts ~/ansible/server/playbooks/rpsetup.yml -u <ansible_user> -K

Roadmap
-------

- Consider revising the Context for the default user accounts to give a list of default accounts to delete.
- Allow overriding of the default_usr variable at a playbook, inventory or run level so we can specificy which account we want to delete

License
-------

BSD

Author Information
------------------

Tristan Findley - Visit https://tfindley.co.uk to find out more.
