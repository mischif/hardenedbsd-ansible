HardenedBSD-Ansible
===================

A collection of playbooks I've written to automate the setup and maintenance of various servers I run - because there's no better sign you yell at computers than to spend weeks trying to automate something you'll do three times max and only takes a couple hours each time.

These playbooks were written to work with HardenedBSD, but should easily translate to FreeBSD or any of its derivatives - just swap out the system/port update commands as necessary.

system_baseline
---------------

The first playbook I run on a new system, meant to set some reasonable defaults I can build on. At a high level, this playbook:

*	Switches from stock FreeBSD OpenSSL to LibreSSL
*	Switches from stock Freebsd OpenSSH to the port compiled against LibreSSL
*	Randomizes the SSH port
*	Enables a pf firewall with a default-deny policy
*	Disables root login over SSH and console
*	Creates a non-root user with passwordless su to root
*	Enables automatic /tmp/ clearing on boot
*	Encrypts the swapfile

Plus a bunch of other little stuff. Add your username and SSH key to `vars/user.yml` and run the playbook:

	$ ansible-playbook system_baseline.yml -i 192.168.56.101, -u root -k -c paramiko

Note: BSDs don't come installed with Python by defaults because they hate freedom, you'll likely have to install it first:

	$ ansible all -m raw -a "env ASSUME_ALWAYS_YES=true pkg install python3" -i 192.168.56.101, -u root -k -c paramiko

But since you'll have to edit sshd_config to allow root login you can probably just install Python then as well.

akkoma_install
--------------

Does what it says on the tin:

*	Installs Akkoma from source
*	Installs PostgreSQL 13 as backing database
*	Installs Caddy as reverse proxy

Enter your details into `vars/akkoma.yml` and you should be ready to go:

	$ ansible-playbook akkoma_install.yml -i 192.168.56.101,

This playbook disables federation and new registrations by default, you can enable them once you've configured your instance how you'd like. This playbook also moves instance configuration into the database, so you'll likely want an admin frontend.

Speaking of frontends, this playbook does not install any by default; you can grab the standard ones like so:

	$ su akkoma -c 'cd ~/akkoma && mix pleroma.frontend install pleroma-fe --ref stable'
	$ su akkoma -c 'cd ~/akkoma && mix pleroma.frontend install admin-fe --ref stable'

akkoma_install_jailed
---------------------

Functionally equivalent to the akkoma_install playbook, but installs Postgres/Akkoma/Caddy into their own separate BSD jails for additional security. The space/maintenance penalty shouldn't be that major since they're thinjails, and part of the reason you pick a BSD over Linux as your base OS is for weirdo features like this, so why not use it?
