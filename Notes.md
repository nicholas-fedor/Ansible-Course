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

Note: Setting up Visual Studio Code is optional, but will enable greater functionality when working across multiple files. I have kept the notes in-line with the course for the time being.

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
ansible-playbook --ask-become-pass remove_apache.yml
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

Copying files to a target node.

- Create a `files` directory:

```console
mkdir files
```

- Create a `default_site.html` file within the `files` directory:

```console
nano /files/default_site.html
```

- Add the following boilerplate HTML:

```default_site.html
<!DOCTYPE html>
<html>
<head>
    <meta charset='utf-8'>
    <meta http-equiv='X-UA-Compatible' content='IE=edge'>
    <title>Hello World!</title>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
</head>
<body>
    <h1>Hello World!</h1>
    <p>Ansible is awesome!</p>
</body>
</html>
```

- Add to Git repo:

```console
git commit -am "Add default_site.html" && git push origin
```

- Open `site.yml`

```console
nano site.yml
```

- Add a play to copy html file to web server:

```site.yml
...
    - name: Add default_site.html to web server
      tags: apache, ubuntu
      ansible.builtin.copy:
        src: default_site.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: "0644"
...
```

- Run the `site.yml` playbook:

```console
ansible-playbook --ask-become-pass site.yml
```

- Check the Ubuntu server to see if the change was successful:

```console
cat /var/www/html/index.html
```

- Visit the webpage using a web browser:

<http://192.168.99.245>

- Update the Git repository:

```console
git commit -am "Add copy html file to web server to site.yml playbook" && git push origin
```

### Part 12. Managing Services

- Open the `site.yml` file:

```console
nano site.yml
```

- Add a play to install Apache on Fedora:

```site.yml
    - name: Install Apache on Fedora
      tags: apache, fedora
      ansible.builtin.dnf:
        name:
          - httpd
      when: ansible_distribution == "Fedora"
```

- Add plays using the `ansible.builtin.service` module:

```site.yml
    - name: Ensure Apache is running on Ubuntu
      tags: apache, ubuntu
      ansible.builtin.service:
        name: apache2
        state: started
      when: ansible_distribution == "Ubuntu"

    - name: Ensure Apache is running on Fedora
      tags: apache, fedora
      ansible.builtin.service:
        name: httpd
        state: started
      when: ansible_distribution == "Fedora"
```

- Run the `site.yml` playbook:

```console
ansible-playbook --ask-become-pass site.yml
```

- Open the `inventory` file:

```console
nano inventory
```

- Duplicate the Fedora server into the `web_servers` group:

```inventory
[web_servers]
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201

[db_servers]
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201
```

- Check the status of the `httpd` service on Fedora:

```console
systemctl status httpd
```

- Reopen the `site.yml` file:

```console
nano site.yml
```

- Add play to change email address in Fedora server's `/etc/httpd/conf/httpd.conf` file:

```site.yml
...
    - name: Change admin email address
      tags: apache, fedora
      ansible.builtin.lineinfile:
        path: /etc/httpd/conf/httpd.conf
        regexp: "^ServerAdmin" # Look for line that starts with ServerAdmin
        line: ServerAdmin somebody@somewhere.net
      when: ansible_distribution == "Fedora"
      register: httpd
...
```

Note: `register` allows storage of state to be registered.

- Add play to restart Apache:

```site.yml
...
    - name: Restart httpd on Fedora
      tags: apache, fedora
      ansible.builtin.service:
        name: httpd
        state: restarted
      when: httpd.changed
...
```

- Run the `site.yml` playbook:

```console
ansible-playbook --ask-become-pass site.yml
```

- Update the Git repository:

```console
git commit -am "Update inventory and add plays to site.yml playbook" && git push origin
```

### Part 13. Adding System Users

- Open the `site.yml` playbook:

```console
nano site.yml
```

- Create a `Create user` play just after the `Install updates for Ubuntu`:

```site.yml
...
    - name: Create user
      tags: always
      ansible.builtin.user:
        name: simone
        groups: root
...
```

- Run the `site.yml` playbook:

```console
ansible-playbook --ask-become-pass site.yml
```

- Verify the user was created on the Fedora and Ubuntu servers:

```console
cat /etc/passwd | grep simone
```

#### Update Ubuntu server to allow Simone to run root commands without a password

- Login to root account:

```console
sudo -s
```

- Create a sudoers file for Simone:

```simone
nano /etc/sudoers.d/simone
```

- Add the following:

```simone
simone ALL=(ALL) NOPASSWD: ALL
```

- Update the file permissions:

```console
chmod 440 /etc/sudoers.d/simone
```

- Repeat on the Fedora server as root:

```console
echo "simone ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/simone && chmod 440 /etc/sudoers.d/simone
```

- Copy `ansible.pub` SSH Key from Ubuntu workstation:

```console
cat ~/.ssh/ansible.pub
```

- Login into servers as Simone:

```console
sudo -su simone
```

- Create a `~/.ssh` directory with appropriate security settings:

```console
mkdir ~/.ssh && chmod 700 ~/.ssh
```

- Add the SSH key to an `authorized_keys` file:

```console
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFsT4j5I7WtfiqNa9hWZ3qOF91CmFf4OdT4bTaV3q7pI nick@udemy-ansible-ubuntu-00" > ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
```

- Update `ansible.cfg` with `remote_user = simone`:

```console
echo "remote_user = simone" >> ansible.cfg
```

- Run the `site.yml` playbook to test the addition of the user configuration:

```console
ansible-playbook site.yml
```

Note: The `--ask-become-pass` flag should not be needed anymore with the Sudoers configuration added for the `simone` user.

This should run normally without any issues.

- Open the `site.yml` playbook:

```console
nano site.yml
```

- Add `Add Sudoers file for Simone` play to `site.yml`:

```site.yml
...
    - name: Add Sudoers file for Simone
      tags: always
      ansible.builtin.copy:
        src: sudoer_simone
        dest: /etc/sudoers.d/simone
        owner: root
        group: root
        mode: "0440"
...
```

- Add `Add SSH key for Simone` play to `site.yml`:

```site.yml
...
    - name: Add SSH key for Simone
      tags: always
      ansible.posix.authorized_key:
        user: simone
        key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFsT4j5I7WtfiqNa9hWZ3qOF91CmFf4OdT4bTaV3q7pI nick@udemy-ansible-ubuntu-00"
...
```

- Create `simone` Sudoers file for Ansible to copy:

```console
echo "simone ALL=(ALL) NOPASSWD: ALL" > ~/Documents/Ansible/files/sudoer_simone
```

- Apply the configuration using the `site.yml` playbook:

```console
ansible-playbook site.yml
```

- Update the Git repository:

```console
git commit -am "Added plays for creating bootstrap user" && git push origin
```

## Section 5: Managing Multiple Servers with Ansible

### Part 14. Implementing Server Roles

- Rename the current `site.yml` to `site_before_lesson_14.yml`

- Create a new `site.yml` with the following:

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

- hosts: all
  become: true
  roles:
    - base

- hosts: db_servers
  become: true
  roles:
    - db_servers

- hosts: web_servers
  become: true
  roles:
    - web_servers
```

- Create a `roles` directory:

```console
mkdir roles
```

- Create `base`, `web_servers`, and `db_servers` subdirectories within the `roles` directory:

```console
mkdir ./roles/base ./roles/web_servers ./roles/db_servers
```

- Create `files` and `tasks` subdirectories within `base`, `web_servers`, and `db_servers` subdirectories:

```console
mkdir ./roles/base/files ./roles/base/tasks && \
mkdir ./roles/web_servers/files ./roles/web_servers/tasks && \
mkdir ./roles/db_servers/files ./roles/db_servers/tasks
```

- Create `main.yml` within `./roles/base/tasks/` directory with the following:

```./roles/base/tasks/main.yml
- name: Create user
  tags: always
  ansible.builtin.user:
    name: simone
    groups:
      - root

- name: Add sudoers file for simone
  tags: always
  ansible.builtin.copy:
    src: sudoer_simone
    dest: /etc/sudoers.d/simone
    owner: root
    group: root
    mode: "0440"

- name: Add SSH key for simone user
  tags: always
  ansible.posix.authorized_key:
    user: simone
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFsT4j5I7WtfiqNa9hWZ3qOF91CmFf4OdT4bTaV3q7pI nick@udemy-ansible-ubuntu-00"
```

- Move the `sudoer_simone` file to `./roles/base/files/`:

```console
mv ./files/sudoer_simone ./roles/base/files
```

- Create a `main.yml` file in `./roles/db_servers/tasks/` with the following:

```./roles/db_servers/tasks/main.yml
- name: Install MariaDB
  ansible.builtin.dnf:
    name:
      - mariadb
  when: ansible_distribution == "Fedora"
```

- Create a `main.yml` file in `./roles/web_servers/tasks/` with the following:

```./roles/web_servers/tasks/main.yml
- name: Install Apache on Ubuntu
  tags: apache, ubuntu
  ansible.builtin.apt:
    name:
      - apache2
      - libapache2-mod-php
  when: ansible_distribution == "Ubuntu"

- name: Install Apache on Fedora
  tags: apache, fedora
  ansible.builtin.dnf:
    name:
      - httpd
  when: ansible_distribution == "Fedora"

- name: Ensure Apache service is running on Ubuntu
  tags: apache, ubuntu
  ansible.builtin.service:
    name: apache2
    state: started
  when: ansible_distribution == "Ubuntu"

- name: Ensure Apache service is running on Fedora
  tags: apache, fedora
  ansible.builtin.service:
    name: httpd
    state: started
  when: ansible_distribution == "Fedora"
```

- Update `inventory` file by removing Fedora from `web_servers` group and adding both it and the Ubuntu server to a new `base` group:

```inventory
[base]
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201

[web_servers]
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245

[db_servers]
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201
```

- Update Git repo:

```console
git commit -am "Update to roles-based playbook structure" && git push origin
```

### Part 15. Taking Advantage of Host Variables

Allows different sets of variables to be applied to hosts.

- Update the `./roles/web_servers/tasks/main.yml` file:

```./roles/web_servers/tasks/main.yml
- name: Install Apache on web servers
  tags: apache
  ansible.builtin.package:
    name:
      - "{{ apache_package }}"

- name: Ensure Apache service is running
  tags: apache
  ansible.builtin.service:
    name: "{{ apache_service }}"
    state: started

- name: Change admin email address
  tags: apache, fedora
  ansible.builtin.lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: "^ServerAdmin"
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "Fedora"
  register: apache

- name: Restart Apache
  ansible.builtin.service:
    name: "{{ apache_service }}"
    state: restarted
  when: apache.changed
```

Instead of referring to distribution-specific package managers, use the `ansible.builtin.package` module.
Also, use the variables, such as `"{{ apache_package }}"` instead of `apache2`.

- Create a `host_vars` directory:

```console
mkdir /home/nick/Documents/Ansible/host_vars
```

- Create configuration files for each host in the `host_vars` directory:

```console
touch ./host_vars/192.168.99.201.yml ./host_vars/192.168.99.245.yml
```

- Update each file with the respective variable definitions:

```192.168.99.201.yml
# Fedora Server VM Udemy-Ansible-Fedora-02
apache_package: httpd
apache_service: httpd
```

```192.168.99.245.yml
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
apache_package: apache2
apache_service: apache2
```

Note: It's preferable to use hostnames instead of IP addresses for defining hosts.

- Remove the `base` group from the `inventory` file:

```inventory
[web_servers]
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245

[db_servers]
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201
```

- Run the `site.yml` file to test the playbook:

```console
ansible-playbook site.yml
```

There should be no errors.

- Setup `notify` within `./roles/web_servers/tasks/main.yml` by modifying the `Change admin email address` play:

```./roles/web_servers/tasks/main.yml
- name: Change admin email address
  tags: apache, fedora
  ansible.builtin.lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: "^ServerAdmin"
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "Fedora"
  notify: restart_apache
```

`register` results in the `Restart Apache` play right away; however, `notify` waits until later to run the play.

- Remove the `Restart Apache` play from `./roles/web_servers/tasks/main.yml` and instead create a new subdirectory `./roles/web_servers/handlers`:

```console
mkdir /home/nick/Documents/Ansible/roles/web_servers/handlers
```

- Create a new handler `Restart_Apache` in `./roles/web_servers/handlers/main.yml`:

```./roles/web_servers/handlers/main.yml
- name: Restart_apache
  ansible.builtin.service:
    name: "{{ apache_service }}"
    state: restarted
```

- Run `site.yml` to test the changes:

```console
ansible-playbook site.yml
```

It's unlikely there will be any changes made.

- To induce a change, update the admin email to `somebody@somewhere.com` in the `./roles/web_servers/tasks/main.yml` file:

```main.yml
...
line: ServerAdmin somebody@somewhere.com
...
```

- Re-add the Fedora server back to the `web_servers` group in the `inventory` file:

```inventory
[web_servers]
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201

[db_servers]
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201
```

- Run the `site.yml` playbook again to push the change:

```console
ansible-playbook site.yml
```

The output should reflect changes were made to the Fedora server.

Note: The `notify` feature allows multiple changes to occur to a host; however, these changes will only trigger the single follow-on task, as opposed to repeatedly triggering the same follow-on task for each change.

- Update the course Git repository:

```console
git commit -am "Add host variables and a handler" && git push origin
```

### Part 16. Creating Templates

A template is a text file that can be customized with certain criteria, including using variables for various things.

- Examine the `/etc/ssh/sshd_config` file and take note of the `PasswordAuthentication` variable:

```console
nano /etc/ssh/sshd_config
```

Note: Root privileges are needed if this file needs to be edited. Otherwise, it is read-only.

- Copy `/etc/ssh/sshd_config` to the Ansible directory with the name `sshd_config_ubuntu`:

```console
cp /etc/ssh/sshd_config sshd_config_ubuntu
```

Note:
If the `openssh` package is not installed, then this will not be available.
Including the file contents here for future availability:

```ssd_config_ubuntu
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin prohibit-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem sftp /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
# X11Forwarding no
# AllowTcpForwarding no
# PermitTTY no
# ForceCommand cvs server
```

- Copy the Fedora server's file to the workstation:

First create a copy on the Fedora server in the user directory:

```console
sudo cp /etc/ssh/sshd_config /home/nick
```

Update the file permissions:

```console
sudo chown nick sshd_config
```

And finally use `scp` on the workstation to copy the file over:

```console
scp 192.168.99.201:/etc/ssh/sshd_config .
```

For reference:

```sshd_config_fedora
# $OpenBSD: sshd_config,v 1.104 2021/07/02 05:11:21 dtucker Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

# To modify the system-wide sshd configuration, create a  *.conf  file under
#  /etc/ssh/sshd_config.d/  which will be automatically included below
Include /etc/ssh/sshd_config.d/*.conf

# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
AuthorizedKeysFile .ssh/authorized_keys

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to no to disable s/key passwords
#KbdInteractiveAuthentication yes

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no
#KerberosUseKuserok yes

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no
#GSSAPIEnablek5users no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin prohibit-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
# WARNING: 'UsePAM no' is not supported in Fedora and may cause several
# problems.
#UsePAM no

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
#X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# override default of no subsystems
Subsystem sftp /usr/libexec/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
# X11Forwarding no
# AllowTcpForwarding no
# PermitTTY no
# ForceCommand cvs server
```

- Rename the files to use the Jinja2 `j2` file extension:

```console
mv sshd_config.ubuntu sshd_config_ubuntu.j2 && \
mv sshd_config.fedora sshd_config_fedora.j2
```

- Make a `templates` subdirectory under `./roles/web_servers`:

```console
mkdir ./roles/web_servers/templates
```

- Move the templates into the newly created directory:

```console
mv sshd_config* ./roles/web_servers/templates
```

- Uncomment `PasswordAuthentication` and add a `passwd_auth` variable within each `sshd_config` file:

```sshd_config
...
PasswordAuthentication {{ passwd_auth }}
...
```

- Declare the `passwd_auth` variable in respective host files with the `host_vars` directory:

```./host_vars/192.168.99.201.yml
# Fedora Server VM Udemy-Ansible-Fedora-02
apache_package: httpd
apache_service: httpd
passwd_auth: no
ssh_template_file: sshd_conf_fedora.j2
```

```./host_vars/192.168.99.245.yml
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
apache_package: apache2
apache_service: apache2
passwd_auth: no
ssh_template_file: sshd_conf_ubuntu.j2
```

- Update the course Git repository:

```console
git commit -am "Add templates for SSH" && git push origin
```

## Section 6: Exploring Additional Ansible Features

### Part 17. Exploring the Galaxy

Nearing the end of the course.
Addressing topics that didn't fit with the other items.
Ansible Galaxy is a repository for roles, collections, etc that are made publicly available from a centralize location.

- Navigate the the website via a web browser: [Ansible Galaxy](https://galaxy.ansible.com/ui/)
- Go to `Roles > Roles` and sort by `Download Count`: [Link](https://galaxy.ansible.com/ui/standalone/roles/?page=1&page_size=10&sort=-download_count>)
- Search for `Apache`: [Link](hhttps://galaxy.ansible.com/ui/standalone/roles/?page=1&page_size=10&sort=-download_count&keywords=apache)
- Select the `geerlingguy.apache` role: [Link](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/apache/)
- Copy the command `ansible-galaxy role install geerlingguy.apache`
- Enter the pasted command into the terminal and run it
- Run `ansible-galaxy list` to show installed roles

- Make sure still within the `~/Documents/Ansible` directory
- Update the `Apply web server configuration` play with the `geerlingguy.apache` role in `site.yml`:

```site.yml
- name: Apply web server configuration
  hosts: apache
  become: true
  roles:
    - geerlingguy.apache
```

- Update the `inventory` file as well by changing the `web_servers` role to `apache`:

```inventory
[apache]
# Ubuntu Server VM Udemy-Ansible-Ubuntu-01
192.168.99.245
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201

[db_servers]
# Fedora Server VM Udemy-Ansible-Fedora-02
192.168.99.201
```

- Run the `site.yml` playbook:

```console
ansible-playbook site.yml
```

- Remove a role installed from Ansible Galaxy:

```console
ansible-galaxy remove geerlingguy.apache
```

- Update the Git repository with the changes:

```console
git commit -am "Add gearlingguy.apache role" && git push origin
```

### Part 18. Keeping Secrets

Exploring Ansible Vault for managing secrets.

`ansible-vault` is the CLI command.

- Create a `secret.txt` file:

```console
ansible-vault create secret.txt
```

Expect a prompt for a new Vault password.
Using `ansible` for this course.
A text editor will open (usually the default editor, Vim).
If Vim, enter i (for insert mode), and add `Hello World`.
Then, hit escape, `:`, and `wq` to write and quit.
A new secret.txt file should appear.

- Add the new file to the Git repository:

```console
git commit -am "Add secret.txt" && git push origin
```

- To decrypt the file:

```console
ansible-vault decrypt secret.txt
```

This will decrypt `secret.txt` back into the `Hello World` text.

- To encrypt the existing `secret.txt` file:

```console
ansible-vault encrypt secret.txt
```

Using a password of `ansible`.
This will result in the file becoming encrypted again.

- To view the file without decrypting it:

```console
ansible-vault view secret.txt
```

- To edit the encrypted file:

```console
ansible-vault edit secret.txt
```

- To save the Vault password to a file `.vault_key`:

```console
echo test123 > ~/.vault_key && chmod 600 ~/.vault_key
```

- To encrypt `secret.txt` using the newly created `.vault_key` file:

```console
ansible-vault encrypt secret.txt --vault-password-file ~/.vault_key
```

### Part 19. Ansible in Reverse

### Part 20. Course Closing and Next Steps
