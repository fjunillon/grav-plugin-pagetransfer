# grav-plugin-pagetransfer
admin plugin for Grav environment
# Page Transfer Plugin

The **Page transfer** Plugin is an extension for [Grav CMS](http://github.com/getgrav/grav). It makes things easier to transfer page or list of pages from your development server to the production one using « rsync & sed » command line tools or « sftp » through the phpseclib v3.
It was developed for linux with very basic needs: quick transfer, quick modify beetween two servers. So the design is a bit rough.

## Installation
At this time with this very early version, download the zip-version of this repository . 

### GPM Installation
To install the plugin via the [GPM](https://learn.getgrav.org/cli-console/grav-cli-gpm), through your system's terminal, navigate to the root of your Grav-installation, and enter:
    bin/gpm install pagetransfer

### manual Installation
Unzip it under `/your/site/grav/user/plugins`. Then rename the folder to `pagetranfer`. 

You should now have all the plugin files under /yoursite/grav/user/plugins/pagetransfer

## Configuration
Before configuring this plugin, you should copy the `user/plugins/pagetransfer/pagetransfer.yaml` to `user/config/plugins/pagetransfer.yaml` and only edit that copy.
(see #usage for all options)

## Before using
First of all, as there will be no use of password, you must configure ssh key before using. The basics of this development rely on using the user « www-data » and another of your choice (only for rsync).
The testing config relayed on two raspbian and all clues given are based on. You may adapt to your environments of course.
In all samples after:
- the remote path will be « /var/www/grav/user/pages »
- « myuser » refers to any user you want but it must have a ssh access to remote.

On dev side, generate ssh key:
- sudo -u www-data ssh-keygen -t rsa -b 4096 -f /var/www/.ssh/id_rsa

Copy the public part to the remote using: 
- sudo -u www-data ssh-copy-id -i /var/www/.ssh/id_rsa.pub myuser@host_name_or_ip-forRemote

Test the connexion:
- sudo -u www-data ssh -o Strict

As you need a second user for rsync (www-data is a no-login user), do the same ssh key manipulations for it.

You must also configure sudoers on the remote: 
- add in /etc/sudoers
  www-data ALL=(ALL) NOPASSWD: /usr/bin/rsync myuser ALL=(www-data) NOPASSWD: ALL

Doing this should avoid prompt of password, but still secured as you are using ssh key.

you can have a check at this time by testing on the command line with someting like:

rsync -avz --relative -e "ssh -o StrictHostKeyChecking=no" --rsync-path='sudo -S -u www-data /usr/bin/rsync' /var/www/grav/user/pages/someFileOrDir myuser@host_name_or_ip-forRemote:/var/www/grav/user/pages

if you want to use « sftp », then you have to modify sshd_config on the remote and add this section:

------------------------

Subsystem       sftp    /usr/lib/openssh/sftp-server

Match User www-data

ForceCommand internal-sftp

ChrootDirectory /var/www

AllowTcpForwarding no

X11Forwarding no

AuthorizedKeysFile /var/www/.ssh/authorized_keys

------------------------

Have a check by running on the command line:
- sftp www-data@host_name_or_ip-forRemote

If it works, you will have a prompt like: 
- Connected to host_name_or_ip-forRemote.
sftp> 

## Usage
Go to the pagetransfer plugin in Grav admin and configure needed options. 
- (Needed) Select the tools you want to use 
- (Optional) rsync -> Give a ssh key path for the user you intend to use (not a good way as you have to give read right to the user key file for www-data, prefer using ssh-copy-id)
- (N) rsync -> Give the user to use
- (N) remote address or name
- (N) remote path destination : if you use rsync, it’s a full path (/var/www/grav/user/pages). if you use sftp, it’s a relative path from ChrootDirectory declared in sshd_config (from the config sample, it will be /grav/user/pages)
- (O) rsync/sftp-> You can select or not if you want to transfer subdirectories found.
- (O) rsync/sftp-> You can select or not if you want to clear distant server cache (using grav clearcache command line) after transfer completion
- (O) rsync/sftp -> You can select to use or not the search and replace mecanism. This works only on text files and you can configure as much key/word you want. For « rsync », the « sed » command line will be use, for « sftp », php code will be use.

Now you can go to the « Pages transfer » option in the left Grav menu.
This will show you all pages from all levels that you can transfer. The listbox is multi-select.
Just click copy and… wait the job to complete. You will finally be redirect to a result page showing all commands and results. 

There is an ultimate option you can add to your pagetransfer.yaml manually: « debugon » :) 
- add « debugon: '1' » and the pagetransfer mecanism will add a lot of debug things in the Grav debug console, but it also cut the redirect to the result page.
 
## Credits
- phpseclib Copyright Jim Wigginton https://phpseclib.com

## To Do
- [ ] a lot of things… ;)
