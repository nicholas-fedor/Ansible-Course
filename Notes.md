# Course Notes

## Section 1: Becoming Familiar with Ansible

### Part 1. Course Introduction

Preliminary introduction of instructor and the course.

### Part 2. Setting Up Our Environment

Course test environment setup using Virtualbox.
As I already had it available, I opted to use VMWare Workstation.
Decided to create a full Ubuntu 24.04.1 Gnome Desktop VM as the workstation client and another Ubuntu 24.04.1 VM as the server.

- Install Ansible via Apt:

```console
sudo apt install ansible -y
```

- Create a directory for the repo and move to it:

```console
mkdir ~/Documents/Ansible && cd ~/Documents/Ansible
```

### Part 3. Running Commands with Ansible

- Create Ansible inventory file and add the Ubuntu VM's IP address:

```console
echo 192.168.99.245 > inventory
```

- Confirm able to SSH to server:

```console
ssh nick@192.168.99.245
```

- Generate a SSH key for use between the workstation and server:

```console
ssh-keygen
```

Follow the prompts and create a new key `home/nick/.ssh/ansible`.
Note: You must use the full path.

- Copy the key to the server:

```console
sudo ssh-copy-id -i /home/nick/.ssh/ansible.pub nick@192.168.99.245
```

Note: -i flag is used to specify the file.

- Test Ansible with Ping command:

```console
ansible all --key-file ~/.ssh/ansible -i ~/Documents/Ansible/inventory -m ping
```

- Add Ansible configuration file:

This allows you to avoid having to specify everytime you run Ansible commands.

```console
nano ansible.cfg
```

Contents:

```ansible.cfg
[defaults]
inventory = ~/Documents/Ansible/inventory
private_key_file = ~/.ssh/ansible
```

- Test the configuration is working:

```console
ansible all -m ping
```

- Show the hosts available from the inventory file:

```console
ansible all --list-hosts
```

- Show all available details:
  Note: Running this with the `all` argument will result in Ansible displaying the details of all hosts.

```console
ansible all -m gather_facts
```

### Part 4. Setting Up Version Control

VCS used in this course are Git and GitHub.
Basic configuration is shown.
I opted to create and add a GPG key for the Ubuntu Desktop workstation to my GitHub account as best practice.
In addition, I installed GitHub's CLI tool to help with authentication and setup with Git.

- Install Git:

```console
sudo apt install git -y
```

- Install GitHub CLI:

```console
sudo apt install gh -y
```

- Create a GPG Key:

```console
gpg --full-generate-key
```

Follow the prompts. The defaults are perfectly fine.
As this is not a permanent key, I did not add a passphrase.

- Obtain the GPG Key sec ID:

```console
gpg --list-secret-keys --keyid-format=long
```

- Export the GPG to add to GitHub account:
  Use the part after "ed25519/" from the previous command output.

```console
gpg --armor --export [insert gpg key info here]
```

Example: `gpg --armor --export EB769FF4668752A8`

- Add GPG key to Git configuration:

```console
git config --global user.signingkey EB769FF4668752A8
```

- Add name to Git configuration:

```console
git config --global user.name "Nick Fedor"
```

- Add email to Git configuration:

```console
git config --global user.email 71477161+nicholas-fedor@users.noreply.github.com
```

- Enable Git to use GPG key:

```console
git config --global commit.gpgsign true
```

- Enable Bash to have access to GPG key on start:

```console
[ -f ~/.bashrc ] && echo -e '\nexport GPG_TTY=$(tty)' >> ~/.bashrc
```

- Use GitHub cli to authenticate with GitHub:

```console
gh auth login
```

Follow the prompts and login via HTTPS.

- Use GitHub cli to setup Git:

```console
gh auth setup-git
```

- Configure Git to use `Main` as the default branch:

```console
git config --global init.defaultBranch Main
```

- Initialize the Git repository:

```console
git init
```

- Stage the current files to the repository:

```console
git add .
```

- Commit the files to the repository:

```console
git commit -m "Add files to the new repo"
```

- Use GitHub cli to create the GitHub repo:

```console
gh repo create
```

Created a README.md and added it to the repo.

- Push changes to the GitHub repository:

```console
git push origin
```

## Section 2: Running Commands and Tasks

### Part 5. Running Elevated Commands

- Attempt to run `apt update` on server without elevation:

```console
ansible all -m apt -a update_cache=true
```

This will not work.

- Add `--become --ask-become-pass` to enable elevation:

```console
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

- To install a package (tested with Git) via Apt to the server:

```console
ansible all -m apt -a name=git --become --ask-become-pass
```

- To install/update the latest version of a package:

```console
ansible all -m apt -a "name=git state=latest" --become --ask-become-pass
```

- To run "apt dist-upgrade" on the server:

```console
ansible all -m apt -a upgrade=dist --become --ask-become-pass
```

### Part 6. Writing Our First Playbook

Getting into using Ansible as intended.

- Create a playbook to install Apache server:

```console
nano install_apache.yml
```

Content:

```install_apache.yml
---
- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    ansible.builtin.apt:
      name: apache2
```

All hosts are referenced as only one host is currently configured via the inventory.
Command elevation is enabled via `become: true`.
The task name has no impact on execution; however, it's a good idea to use a descriptive name for logging purposes.
The module is the `apt` built-in Ansible module.
The package to install is `apache2`.

- To run the playbook:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

The output from running playbooks is different from the earlier commands in that it omits the verbose output and simply outputs the status of the playbook's tasks.

- To confirm the status of Apache2 on the server:

Using a shell on the server:

```console
systemctl status apache2
```

If Apache2 is running, then a default starter webpage can also be accessed by navigating to the server via a web browser.
<http://192.168.99.245>

- Add another task to the `install_apache.yml` playbook:

```console
nano install_apache.yml
```

Add additional task before `install apache2 package`.
This is equivalent to `apt update` prior to installing packages.

```install_apache.yml
  - name: update repository index
    ansible.builtin.apt:
      update_cache: yes
```

- Run the playbook again:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

The output will show the additional task `update repository index` and indicate `changed`.

- Add an additional play (task) to the playbook:

```console
nano install_apache.yml
```

Add after install apache2 package

```install_apache.yml
  - name: install support for php
    ansible.builtin.apt:
      name: libapache2-mod-php
```

- Run the playbook again:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

The output will show the results of the plays and the play recap.

- Adding an option to a play to ensure the latest package is installed:

```console
nano ansible_playbook.yml
```

Add the following after `name:apche2`
Ensure it's indented on the same level as the prior `name` key.

```ansible_playbook.yml
      state: latest
```

Do the same for the `install support for php` play.

- Again, run the playbook to test:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

It's likely only `update repository index` will reflect a `changed` status.

- To reverse the process and remove installed packages:

Make a copy of the install playbook:

```console
cp install_apache.yml remove_apache.yml
```

- Update `remove_apache.yml`:

```console
nano remove_apache.yml
```

- Change the names of the plays and the `state` to `absent` for each package:

```remove_apache.yml
---
- hosts: all
  become: true
  tasks:

  - name: update repository index
    ansible.builtin.apt:
      update_cache: yes

  - name: remove apache2 package
    ansible.builtin.apt:
      name: apache2
      state: absent

  - name: remove support for php
    ansible.builtin.apt:
      name: libapache2-mod-php
      state: absent
```

- Run the `remove_apache.yml` playbook:

```console
ansible-playbook --ask-become-pass remove_apache.yaml
```

The output should show each play is run and an indication of four (4) ok's and three (3) changes.

- Check the server again to confirm Apache2 is no longer installed:

```console
systemctl status apache2
```

- Update the Git and GitHub repository:

```console
git add . && \
git commit -m "Add Apache2 installation and removal playbooks" && \
git push origin
```

### Part 7. Dealing with Mixed Linux Environments

Using a Fedora server VM (IP: 192.168.99.201) to simulate using a mixed Linux environment.

- Copy the workstation's SSH key to the Fedora server:

```console
ssh-copy-id -i ~/.ssh/ansible.pub nick@192.168.99.201
```

- Test SSH connectivity to the Fedora server:

```console
ssh nick@192.168.99.201
```

- Add the Fedora server to the Ansible inventory file:

```console
echo 192.168.99.201 >> inventory
```

- Check Git status:

```console
git status
```

- Show changes to the `inventory` file:

```console
git diff inventory
```

- Commit the changes to the `inventory` file:

```console
git commit -am "Added Fedora server"
```

Note: Using the `-a` flag adds files that are already being watched by Git.

- Push the changes to GitHub:

```console
git push origin
```

- Run the `install_apache.yml` playbook:
  Note: This will run against both the Ubuntu and Fedora servers.

```console
ansible-playbook --ask-become-pass install_apache.yml
```

A failure should be expected with the Fedora server, as it does not use the Apt package manager.

- Update the `install_apache.yml` playbook:

```console
nano install_apache.yml
```

Add the conditional statement `when: ansible_distribution == "Ubuntu"` to each play.

Example:

```install_apache.yml
---
- hosts: all
  become: true
  tasks:

  - name: update repository index
    ansible.builtin.apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install apache2 package
    ansible.builtin.apt:
      name: apache2
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install support for php
    ansible.builtin.apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"
```

- Run the playbook:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

It is expected that the Ansible will skip the Fedora server entirely when running the playbook.

- Example of how Ansible is able to gather facts about a host:

```console
ansible all -m gather_facts --limit 192.168.99.201
```

This will limit the output to that generated from the Fedora server.
Note: If the output is further filtered by using `grep ansible_distribution`, then `Fedora` will be listed.

- Update the `install_apache.yml` playbook to include plays for the Fedora server:

```console
nano install_apache.yml
```

Copy the three plays and paste a new set below the originals.
Then, update the new set to use `dnf` instead of `apt` Ansible modules and `Fedora` instead of `Ubuntu` in the conditional for the distribution.
Change the Fedora server to install `httpd` and `php` instead of `apache2` and `libapach2-mod-php`, respectively.

Example:

```install_apache.yml
---
- hosts: all
  become: true
  tasks:

  - name: update repository index
    ansible.builtin.apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install apache2 package
    ansible.builtin.apt:
      name: apache2
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install support for php
    ansible.builtin.apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: update repository index
    ansible.builtin.dnf:
      update_cache: yes
    when: ansible_distribution == "Fedora"

  - name: install httpd package
    ansible.builtin.dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "Fedora"

  - name: install support for php
    ansible.builtin.dnf:
      name: php
      state: latest
    when: ansible_distribution == "Fedora"
```

- Run the playbook:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

This should reflect three (3) skips for each host, but each should have successful installs of the respective packages.

- Add and commit the changes to Git, and upload them to GitHub:

```console
git commit -am "Add httpd and PHP to Fedora server" && git push origin
```

## Section 3: Organizing our Repository

### Part 8. Refactoring and Simplifying Our Playbook

- Open the `install_apache.yml` playbook:

```console
nano install_apache.yml
```

- Combine the plays to install Apache and PHP for each distribution.

Example:

```install_apache.yml
---
- hosts: all
  become: true
  tasks:

  - name: update repository index
    ansible.builtin.apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install apache2 and libapache2-mod-php packages
    ansible.builtin.apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: update repository index
    ansible.builtin.dnf:
      update_cache: yes
    when: ansible_distribution == "Fedora"

  - name: install httpd and php packages
    ansible.builtin.dnf:
      name:
        - httpd
        - php
      state: latest
    when: ansible_distribution == "Fedora"
```

- Run the updated `install_apache.yml` playbook:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

The expectation is that the playbook should run with no errors.

- Examine the status of `httpd` on the Fedora server:

```console
systemctl status http
```

It should show as installed; however, will not be running.
This will be addressed later in the course.

- Refactor the `install_apache.yml` playbook even further:

```console
nano install_apache.yml
```

Instead of separate plays for updating the repository indexes, `update_cache: yes` can be used within the installation plays.

Example:

```install_apache.yml
---
- hosts: all
  become: true
  tasks:

  - name: install apache2 and libapache2-mod-php packages
    ansible.builtin.apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install httpd and php packages
    ansible.builtin.dnf:
      name:
        - httpd
        - php
      state: latest
      update_cache: yes
    when: ansible_distribution == "Fedora"
```

- Run the updated `install_apache.yml` playbook:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

The expectation is that the playbook should run with no errors.

- It is possible to bring the `install_apache.yml` playbook down to a single play:

```console
nano install_apache.yml
```

Change the modules from `apt` and `dnf` to `package`.
Because the package names are different for Ubuntu and Fedora, use host variables.

Example:

```install_apache.yml
---
- hosts: all
  become: true
  tasks:

  - name: install apache2 and php
    ansible.builtin.package:
      name:
        - "{{ apache_package }}"
        - "{{ php_package }}"
      state: latest
      update_cache: yes
```

This also requires updating the `inventory` file.

Example:

```inventory
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245 apache_package=apache2 php_package=libapache2-mod-php
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201 apache_package=httpd php_package=php
```

- Run the updated `install_apache.yml` playbook:

```console
ansible-playbook --ask-become-pass install_apache.yml
```

The expectation is that the playbook should run with no errors.

- Update the Git repository:

```console
git commit -am "Refactor install_apache.yml playbook and update inventory" && git push origin
```

### Part 9. Creating an Inventory for Ansible

Learning about inventory groups.

- Open the `inventory` file:

```console
nano inventory
```

- Remove the variables and add groups `web_servers` and `db_servers`, for example:

```inventory
[web_servers]
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245

[db_servers]
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201
```

Note: It is possible to have each host be a member of multiple groups, i.e. the Fedora server in both the `web_servers` and `db_servers` groups.

- Create a new `site.yml` playbook:

```site.yml
nano site.yml
```

Note the `hosts:` section is used to specify the groups.
Use the `pre_tasks` section to ensure these plays are run first.
This playbook can be refactored; however, it will be kept relatively simple for purposes of the course.

Example:

```site.yml
---
- hosts: all
  become: true
  pre_tasks:

  - name: install updates for Fedora
    ansible.builtin.dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "Fedora"

  - name: install updates for Ubuntu
    ansible.builtin.apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

- hosts: web_servers
  become: true
  tasks:

  - name: install apache on web servers
    ansible.builtin.apt:
      name:
        - apache2
        - libapache2-mod-php
    when: ansible_distribution == "Ubuntu"

- hosts: db_servers
  become: true
  tasks:

  - name: install mariadb package on db servers
    ansible.builtin.dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "Fedora"
```

- Run the `site.yml` playbook:

```console
ansible-playbook --ask-become-pass site.yml
```

This playbook is expected to run without issues and install MariaDB on the Fedora server.

- Update Git with the changes:

```console
git commit -am "Add site.yml playbook and updated inventory file to include groups and remove variables" && git push origin
```

### Part 10. Adding Tags

Using tags is a way to run a play without having to run all plays in a playbook.

- Open `site.yml`:

```console
nano site.yml
```

- Add `tags: always` tags to `Install updates` plays for both Fedora and Ubuntu.
- Add `tags: apache, ubuntu` tag to `Install apache` play.
- Add `tags: db, fedora` tag to `Install site database` play.

Example:

```site.yml
---
- name: Install updates
  hosts: all
  become: true
  pre_tasks:

    - name: Install updates for Fedora
      tags: always
      ansible.builtin.dnf:
        update_only: true
        update_cache: true
      when: ansible_distribution == "Fedora"

    - name: Install updates for Ubuntu
      tags: always
      ansible.builtin.apt:
        upgrade: dist
        update_cache: true
      when: ansible_distribution == "Ubuntu"

- name: Install site web server
  hosts: web_servers
  become: true
  tasks:

    - name: Install apache
      tags: apache, ubuntu
      ansible.builtin.apt:
        name:
          - apache2
          - libapache2-mod-php
      when: ansible_distribution == "Ubuntu"

- name: Install site database
  tags: db, fedora
  hosts: db_servers
  become: true
  tasks:

    - name: Install mariadb
      ansible.builtin.dnf:
        name: mariadb
      when: ansible_distribution == "Fedora"
```

- Remove MariaDB from the Fedora server:

```console
sudo yum remove mariadb -y
```

- Run `site.yml` using the `--tags db` tag:

```console
ansible-playbook --tags db --ask-become-pass site.yml
```

This is expected to install MariaDB on the Fedora server specifically due to the use of the `db` tag.

- Repeat, but using the `fedora` tag instead:

```console
ansible-playbook --tags fedora --ask-become-pass site.yml
```

This is expected to run the plays tagged with `fedora`.

- Repeat, but using the `apache` tag instead:

```console
ansible-playbook --tags apache --ask-become-pass site.yml
```

This is expected to run the plays tagged with `apache`.

- Update Git repository:

```console
git commit -am "Add tags to site.yml" && git push origin
```

## Section 4: System Administration with Ansible

### Part 11. Copying Files with Ansible

### Part 12. Managing Services

### Part 13. Adding System Users

## Section 5: Managing Multiple Servers with Ansible

### Part 14. Implementing Server Roles

### Part 15. Taking Advantage of Host Variables

### Part 16. Creating Templates

## Section 6: Exploring Additional Ansible Features

### Part 17. Exploring the Galaxy

### Part 18. Keeping Secrets

### Part 19. Ansible in Reverse

### Part 20. Course Closing and Next Steps
