Ansible Prep
=========

This role is used for first-time deployment of an Ansible user, and lockdown/deletion of the stock system account

BEFORE USING THIS ROLE: Read the requirements section as this role deletes a user account


Preface
-------

This role has evolved from an original requirement to deploy / manage multiple Raspberry Pi systems. Raspbian (by default) creates a user called 'pi' with the password 'raspberry'. We extend this model to cover all (supported) operating systems and assume we will have a preset account that will only be used to execute this one role, and then after that the account will be deleted.

This role utilises that existing account to deploy a new user 'ansible_usr' to the system with the password 'ansible_pwd'. An RSA public key 'ansible_key' is also created for that user.


A second run of the role (this time as the ansible user that we just created) will destroy the default account that was used during the first run. This is determined through a preset list of accounts that exist in the context section of the var folder.


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
- ansible_usr: Ansible account username
- ansible_pwd: Desired password, this must be encrypted! - use the following command to generate this: $ python -c 'import crypt; print crypt.crypt("This is my Password", "$1$SomeSalt$")'
- ansible_key: The public half of the SSH keypair generated: use the following command to generate this: $ ssh-keygen
- default_usr: This is set across various OS-specific var files and uses the os name (lowercase). Manual setting of this variable is not yet supported but planned for a future redesign.

It is suggested that these variables exist in your inventory file instead of in the playbook so they can be utilised on multiple runs without the duplication of information. Furthermore as these variables should probably apply to the whole environment  (as covered by the inventory) this would be the most logical place to store then. Please see Example Playbook / Example Inventory

Preset Variables
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

CentOS 7 - just use the minimal install disk
CentOS 8 - requires you to use the full fat disk to install this, so you will need to choose Minimum on the correct screen. 
Debian 9 - install the minimum or network install. Do not select to install System tools but you must choose to set the system up as an SSH server otherwise you won’t be able to connect. Once installed, you must install the package ‘python’ as it is not included by defailt
Debian 10 - install the minimum or network install. Do not select to install System tools but you must choose to set the system up as an SSH server otherwise you won’t be able to connect. 
Raspbian - You can use the Minimum install image. No additional packages are required.

Simply run: # apt install python 

There are no direct Ansible or System dependencies. The Role will install any basic Ansible requirements such as python modules.

Example Inventory
-----------------

      [all:vars]
      ansible_usr: bart
      ansible_pwd: $1$SomeSimp$f107RFQFLQDHVqKfo335i.
      ansible_key: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlA6OitiXIwhWno5M1hxQVWueCibVm5Y/dEw3xxbCXxwpB9DhPdqXGHabtVWjBqEqSLK/cjeuqm3x/qsNHZq3OFgD+1PQnJsT2QWFeajLHBK3fEL9LPf6b1UIbCNNFwW12U2wMyFAsMtfBbcABaME7dYrzT2jB/EmgLVycUGmQsTh4hEqbaB3uXXtkyGLRTB9RInHpQtbTRPdmPPv+i01yAnBon0bqWJKL7SsSsBidqy9qaPumrTOEUXUl5CLkc/k2cwt34CJNYzDUUHAp642IjfjQnchTUcSk7ydrKuoT0MF9DQaMRdvx7tbxsO+T9oZ4CugNIKEVnALbOy8hRNHB user@machine.lan


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      become: yes
      gather_facts: true
      roles:
        - role: '~/ansible/roles/ansible-prep'
      vars:
        ansible_usr: bart
        ansible_pwd: $1$SomeSimp$f107RFQFLQDHVqKfo335i.
        ansible_key: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlA6OitiXIwhWno5M1hxQVWueCibVm5Y/dEw3xxbCXxwpB9DhPdqXGHabtVWjBqEqSLK/cjeuqm3x/qsNHZq3OFgD+1PQnJsT2QWFeajLHBK3fEL9LPf6b1UIbCNNFwW12U2wMyFAsMtfBbcABaME7dYrzT2jB/EmgLVycUGmQsTh4hEqbaB3uXXtkyGLRTB9RInHpQtbTRPdmPPv+i01yAnBon0bqWJKL7SsSsBidqy9qaPumrTOEUXUl5CLkc/k2cwt34CJNYzDUUHAp642IjfjQnchTUcSk7ydrKuoT0MF9DQaMRdvx7tbxsO+T9oZ4CugNIKEVnALbOy8hRNHB user@machine.lan


First Run
---------

To run the playbook :
Raspberry Pi (uses sudo): (password: raspberry)
ansible-playbook -i ~/ansible/server/inventory/hosts ~/ansible/server/playbooks/ansible-prep.yml -u pi -k

Debian (uses su):
ansible-playbook -i ~/ansible/server/inventory/dev ~/ansible/server/playbooks/ansible-prep.yml --become-method su -u <user> -k -K

CentOS (uses sudo):
ansible-playbook -i ~/ansible/server/inventory/dev ~/ansible/server/playbooks/ansible-prep.yml -u <user> -k

Second Run
----------

(ansible auth via SSH keypair. SUDO via preset password)
$ ansible-playbook --private-key ansible/server/ssh/id_rsa -i ~/ansible/server/inventory/hosts ~/ansible/server/playbooks/rpsetup.yml -u <ansible_usr> -K

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
