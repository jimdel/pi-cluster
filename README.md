# Raspberry Pi Cluster Ansible Automation

## What does this stuff do?

This repository contains opinionated playbooks that can be used to prepare a cluster of raspberry pis. The `prep.yml` playbook will update the base software and run an initial update & upgrade. The `intialize.yml` playbook will change the default credentials, lock the default account, create a new sudo enabled user, enable ssh authenticating via a key-pair, install a firewall, and much more. After running this playbook you can remove the `ansible_ssh_pass` entry from the `inventory.yml` file; ssh will now use key-pair authentication.

**Warning: The intialize.yml playbook disables Wi-Fi & Bluetooth**

## Directions


1. Flash RaspianOS Lite onto your SD card using the Raspian Imager
2. Perform Initial Boot
   1. Plug the SD card into the Pi
   2. plug in power
   3. wait 2 minutes
   4. Disconnect power
   5. Remove SD card 
1. Run `ansible-playbook prep-local.yml`
2. Plug the SD card back into the Pi & connect power
3. Test connection to the pi `ping -c 5 <IP ADDRESS>`
4. `ssh pi@<IP ADDRESS>` to log into the pi and then run `sudo rpi-update`
5. Rename `inventory_template.yml` to `inventory.yml`
6. Fill out the inventory file with information about your devices
   1. The attached template is geared toward a cluster, mine has one master and 4 nodes
7. Run `ansible-playbook ping.yml`
8. Run `ansible-playbook prep-remote.yml`
9. Run `ansible-playbook initialize.yml`
10. Edit the `inventory.yml` and remove the `ansible_ssh_password` entry and change the `ansible_ssh_host` to your new username
11. Run `ansible-playbook k3s-prep.yml`
12. Run `k3s-master.yml -K`
    1.  The `BECOME` password prompt is asking for you local machine sudo password so that it can write the master token to a file
13. Run `k3s-nodes.yml`

## Requirements

`ansible` installed on your control node

`sshpass` for initial `ssh` connection until the key-pair is configured:

    Ubuntu
    sudo apt-get install sshpass

    MacOS
    brew install hudochenkov/sshpass/sshpass

`ansible-galaxy collection install community.general`

`ansible-galaxy collection install ansible.posix`


## Useful Commands

SSH to an IP
```
ssh user@0.0.0.0
```

Start a playbook
`ansible-playbook PLAYBOOK_NAME`

Review the inventory JSON
`ansible-inventory --list`
    Use `-i /path/to/inventoryfile` if different from inventory file specified in `ansible.cfg`

Generate SSH keys (save them to the `/keys` dir)
```bash
#!/bin/bash
ssh-keygen -t rsa -b 4096
```

Ansible - pass extra variables when running commands
```
--extra-vars newpassword=12345678 example=lilyhammer
```
## Reference Material

### Videos
- [Network Chuck Pi Cluster - Video](https://www.youtube.com/watch?v=X9fSMGkjtug&t=1058s)
- [The Digital Life Ansible Automation - Video](https://www.youtube.com/watch?v=uR1_hlHxvhc&t=1382s)
- [How to Secure a Raspberry Pi - Video](https://www.youtube.com/watch?v=ukHcTCdOKrc)


### Articles and Docs
- https://www.cyberciti.biz/faq/ansible-apt-update-all-packages-on-ubuntu-debian-linux/
- https://docs.ansible.com/ansible-core/devel/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups
- https://bitsanddragons.wordpress.com/2021/05/27/avoid-typing-ssh-passwords-with-sshpass-on-macos/
- https://github.com/kenfallon/fix-ssh-on-pi
- https://serverfault.com/questions/566762/how-do-i-add-sudo-permissions-to-a-user-created-with-ansible
- https://www.digitalocean.com/community/tutorials/how-to-use-nmap-to-scan-for-open-ports
- https://kofler.info/kernel-4-0-auf-dem-raspberry-pi/
- https://galaxy.ansible.com/weareinteractive/ufw
- https://www.redhat.com/sysadmin/ansible-playbooks-secrets
- https://rancher.com/docs/k3s/latest/en/installation/uninstall/

### Gotchas

If you're like me and had to re-image a pi and start from scratch (and you used the same ip as the first go-round) you may come across this error when attempting to connect to the re-imaged pi(s) 

```

fatal: [master]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@\r\n@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @\r\n@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@\r\nIT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!\r\nSomeone could be eavesdropping on you right now (man-in-the-middle attack)!\r\nIt is also possible that a host key has just been changed.\r\nThe fingerprint for the ECDSA key sent by the remote host is\nSHA256:vm3d/KZcb0Ncgo1H+MIumOwp1KZTWBZZqI7tjjpkybk.\r\nPlease contact your system administrator.\r\nAdd correct host key in /Users/user/.ssh/known_hosts to get rid of this message.\r\nOffending ECDSA key in /Users/user/.ssh/known_hosts:5\r\nPassword authentication is disabled to avoid man-in-the-middle attacks.\r\nKeyboard-interactive authentication is disabled to avoid man-in-the-middle attacks.\r\npi@192.168.1.70: Permission denied (publickey,password).", "unreachable": true}

```

Solution - delete the old entry (entire line) with the ip address of the re-imaged node in `~/.ssh/known_hosts` file. Using `vim` you can highlight over the line in question while in CMD mode and type `dd`


### To - Do's

Make `prep-remote.yml` use a special host group for new pis while also adding them to cluster & node groups