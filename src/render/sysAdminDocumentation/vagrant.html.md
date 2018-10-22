---
title: "Vagrant"
icon: "fas fa-info-circle"
layout: "sysAdminDocumentationTemp"
isSysAdminDocumentation: true
menuIndex: 6
---

The ReDBox team have put together a vagrant box for development use. The vagrant box may also be suitable for spinning up an instance of the system for evaluation purposes.

## Installation instructions

1. Install Vagrant and Unison

If you're using OSX you may wish to follow the instructions further down this page to install the required components using Homebrew. Otherwise, see the [Vagrant ](https://www.vagrantup.com/docs/installation/) and [Unison](https://www.cis.upenn.edu/~bcpierce/unison/) websites for instructions on how to install them for your system.

2. Clone the vagrant-redbox-dev repository from GitHub

`git clone https://github.com/qcif/vagrant-redbox-dev.git`

3. Configure source directory to be mounted on box
Edit the Vagrantfile and set the config.unison.host_folder to the directory you'd like to sync. By default this is setup to be the parent directory so if you structure your directories in the following way then you can skip this step.

RedBox<br>
|<br>
| - vagrant-redbox-dev<br>
|<br>
| - redbox-portal<br>
|<br>

4. Clone the ReDBox Portal project from GitHub into the directory you configured to sync

`git clone https://github.com/redbox-mint/redbox-portal.git`

5. Bring box up

`vagrant up`

Your box should initialise and install all the required development tools

6. Install vagrant unison plugin

`vagrant plugin install vagrant-unison2`

7. Start unison file synching

`vagrant unison-sync-polling`

This will start polling every second your local machine share directory and the directory on the virtual machine and keep them in sync as files change. You may want to do this in another console window as this will stay running until you hit Ctrl+C

8. SSH onto box

`vagrant ssh`

9. Navigate to the redbox-portal directory

`cd source/redbox-portal`

10. Build the portal and start the application

`./runForDev.sh install jit`

This command will launch docker, run yarn to download any required dependencies and build the Angular applications that the portal uses. For subsequent runs you can just start the application using the command

`./runForDev.sh`

## Installing on the Mac via Homebrew

1. [Install Homebrew](http://brew.sh/)

`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

2. Install Vagrant and Unison on local machine

`brew install vagrant`
or
`brew cask install vagrant`

`brew install unison`
