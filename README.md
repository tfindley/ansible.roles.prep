Ansible Prep
=========

This role is used for first-time deployment of an Ansible user, and lockdown/deletion of the stock system account

WARNING: Read the requirements section as this role deletes a user account

Preface
-------

This role has evolved from an original requirement to deploy / manage multiple Raspberry Pi systems. Raspbian by default creates a user called 'pi' with the password 'raspberry'. In the original model of this role we used this account to deploy a new privilaged user and lock down the pi user. This was done on the first run of the role using the pi user.
A second run of the role (this time as the ansible user that we just created) would subsequently destroy the pi account.

Going forward, I have expanded the role to support most common Linux OSes. Using the raspberry pi model we assume the following:
- A user exists on the system with the same name as the distribution (raspberry pi is exempt as it always uses the account pi) i.e: CentOS 7 will have an account called 'centos'. This account will be deleted upon the second run.
- SSH is enabled

Requirements
------------

- An account exists on the system with the same name as the os (this uses the default_usr var). It assumes that if you use CentOS then your default user account will be centos.
WARNING: This account will be deleted! Only use it for the initial setup

- SSH access enabled. On a raspberry pi create a blank file called 'ssh' on the boot partition of the SD Card to enable this at first boot (useful in Headless mode)

You will also need:
- Ansible username
- Ansible user password - Create this using: $ python -c 'import crypt; print crypt.crypt("This is my Password", "$1$SomeSalt$")'
- Ansible user SSH keypair - Create this using: $ ssh-keygen


Role Variables
--------------

- ansible_usr: Ansible account username
- ansible_pwd: Desired password, this must be encrypted! - use the following command to generate this: $ python -c 'import crypt; print crypt.crypt("This is my Password", "$1$SomeSalt$")'
- ansible_key: The public half of the SSH keypair generated: use the following command to generate this: $ ssh-keygen
- default_usr: This is set across various OS-specific var files and uses the os name (lowercase). It is not setable yet.

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

License
-------

BSD

Author Information
------------------

Tristan Findley - Visit https://tfindley.co.uk to find out more.
