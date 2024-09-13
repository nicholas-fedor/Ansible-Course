# Course Notes

## Section 1: Becoming Familiar with Ansible

### Part 1. Course Introduction

Preliminary introduction of instructor and the course.

### Part 2. Setting Up Our Environment

Course test environment setup using Virtualbox.
As I already had it available, I opted to use VMWare Workstation.
Decided to create a full Ubuntu 24.04.1 Gnome Desktop VM as the workstation client and another Ubuntu 24.04.1 VM as the server.

* Install Ansible via Apt:

```console
sudo apt install ansible -y
```

* Create a directory for the repo and move to it:

```console
mkdir ~/Documents/Ansible && cd ~/Documents/Ansible
```

### Part 3. Running Commands with Ansible

* Create Ansible inventory file and add the Ubuntu VM's IP address:

```console
echo 192.168.99.245 > inventory
```

* Confirm able to SSH to server:

```console
ssh nick@192.168.99.245
```

* Generate a SSH key for use between the workstation and server:

```console
ssh-keygen
```

Follow the prompts and create a new key `home/nick/.ssh/ansible`.
Note: You must use the full path.

* Copy the key to the server:

```console
sudo ssh-copy-id -i /home/nick/.ssh/ansible.pub nick@192.168.99.245
```
Note: -i flag is used to specify the file.

* Test Ansible with Ping command:

```console
ansible all --key-file ~/.ssh/ansible -i ~/Documents/Ansible/inventory -m ping
```

* Add Ansible configuration file:

This allows you to avoid having to specify everytime you run Ansible commands.

```console
nano ansible.cfg
```

Contents:

```ansible.cfg
[defaults]
inventory = ~/Documents/Ansible/inventory
private_key_file = ~/.ssh/ansible
````

* Test the configuration is working:

```console
ansible all -m ping
```

* Show the hosts available from the inventory file:

```console
ansible all --list-hosts
```

* Show all available details:
Note: Running this with the `all` argument will result in Ansible displaying the details of all hosts.

```console
ansible all -m gather_facts
```

### Part 4. Setting Up Version Control

VCS used in this course are Git and GitHub.
Basic configuration is shown.
I opted to create and add a GPG key for the Ubuntu Desktop workstation to my GitHub account as best practice.
In addition, I installed GitHub's CLI tool to help with authentication and setup with Git.

* Install Git:

```console
sudo apt install git -y
```

* Install GitHub CLI:

```console
sudo apt install gh -y
```

* Create a GPG Key:

```console
gpg --full-generate-key
```
Follow the prompts. The defaults are perfectly fine.
As this is not a permanent key, I did not add a passphrase.

* Obtain the GPG Key sec ID:

```console
gpg --list-secret-keys --keyid-format=long
```

* Export the GPG to add to GitHub account:
Use the part after "ed25519/" from the previous command output.

```console
gpg --armor --export [insert gpg key info here]
```

Example: `gpg --armor --export EB769FF4668752A8`

* Add GPG key to Git configuration:

```console
git config --global user.signingkey EB769FF4668752A8
```

* Add name to Git configuration:

```console
git config --global user.name "Nick Fedor"
```

* Add email to Git configuration:

```console
git config --global user.email 71477161+nicholas-fedor@users.noreply.github.com
```

* Enable Git to use GPG key:

```console
git config --global commit.gpgsign true
```

* Enable Bash to have access to GPG key on start:

```console
[ -f ~/.bashrc ] && echo -e '\nexport GPG_TTY=$(tty)' >> ~/.bashrc
```

* Use GitHub cli to authenticate with GitHub:

```console
gh auth login
```
Follow the prompts and login via HTTPS.

* Use GitHub cli to setup Git:

```console
gh auth setup-git
```

* Configure Git to use `Main` as the default branch:

```console
git config --global init.defaultBranch Main
```

* Initialize the Git repository:

```console
git init
```

* Stage the current files to the repository:

```console
git add .
```

* Commit the files to the repository:

```console
git commit -m "Add files to the new repo"
```

* Use GitHub cli to create the GitHub repo:

```console
gh repo create
```

Created a README.md and added it to the repo.

* Push changes to the GitHub repository:

```console
git push origin
```

## Section 2: Running Commands and Tasks

### Part 5. Running Elevated Commands

* Attempt to run `apt update` on server without elevation:

```console
ansible all -m apt -a update_cache=true
```
This will not work.

* Add `--become --ask-become-pass` to enable elevation:

```console
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

* To install a package (tested with Git) via Apt to the server:

```console
ansible all -m apt -a name=git --become --ask-become-pass
```

* To install/update the latest version of a package:

```console
ansible all -m apt -a "name=git state=latest" --become --ask-become-pass
```

* To run "apt dist-upgrade" on the server:

```console
ansible all -m apt -a upgrade=dist --become --ask-become-pass
```

### Part 6. Writing Our First Playbook

### Part 7. Dealing with Mixed Linux Environments

## Section 3: Organizing our Repository

### Part 8. Refactoring and Simplifying Our Playbook

### Part 9. Creating and Inventory for Ansible

### Part 10. Adding Tags

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
