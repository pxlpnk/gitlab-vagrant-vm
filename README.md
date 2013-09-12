Gitlab-Vagrant-VM
=================

Description
-----------

Setup a dev environment for Gitlab.

The final product contain all databases set up, working tests and all gems
installed.

Requirements
------------

* [VirtualBox](https://www.virtualbox.org)
* [Vagrant 1.2.x](http://vagrantup.com)
* the NFS packages. Already there if you are using Mac OS, and
  not necessary if you are using Windows. On Linux:

    ```bash
    $ sudo apt-get install nfs-kernel-server nfs-common portmap
    ```
    On OS X you can also choose to use [the (commercial) Vagrant VMware Fusion plugin](http://www.vagrantup.com/vmware) instead of VirtualBox.

* some patience :)

**Note:** Make sure to use Vagrant v1.2.x. Do not install via rubygems.org as there exists an old gem
which will probably cause errors. Instead, grab the latest version from http://downloads.vagrantup.com/.

Installation
------------

Clone the repository:

```bash
$ git clone https://github.com/gitlabhq/gitlab-vagrant-vm
$ cd gitlab-vagrant-vm
```

And install gems and chef's necessary packages:

```bash
$ bundle install
$ bundle exec librarian-chef install
```

Finally, you should be able to use:

```bash
$ vagrant up
```

By default the VM uses 1GB of memory and 1 CPU core. If you want to use more memory or cores you can use the GITLAB_VAGRANT_MEMORY and GITLAB_VAGRANT_CORES environment variables:
```bash
GITLAB_VAGRANT_MEMORY=1536 GITLAB_VAGRANT_CORES=2 vagrant up
```

**Note:**
You can't use a vagrant project on an encrypted partition (ie. it won't work if your home directory is encrypted).

You'll be asked for your password to set up NFS shares.

Once everything is done you can log into the virtual machine to run tests:

```bash
$ vagrant ssh
$ cd /vagrant/gitlabhq/
$ bundle exec rake gitlab:test
```

Start the Gitlab app:
```bash
$ bundle exec foreman start
```

You should also configure your own remote since by default it's going to grab
gitlab's master branch.

```bash
$ git remote add mine git://github.com/me/gitlabhq.git
$ # or if you prefer set up your origin as your own repository
$ git remote set-url origin git://github.com/me/gitlabhq.git
```

Virtual Machine Management
--------------------------

When done just log out with `^D` and suspend the virtual machine

```bash
$ vagrant suspend
```

then, resume to hack again

```bash
$ vagrant resume
```

Run

```bash
$ vagrant halt
```

to shutdown the virtual machine, and

```bash
$ vagrant up
```

to boot it again.

You can find out the state of a virtual machine anytime by invoking

```bash
$ vagrant status
```

Finally, to completely wipe the virtual machine from the disk **destroying all its contents**:

```bash
$ vagrant destroy # DANGER: all is gone
```

Information
-----------

* Virtual Machine IP: 192.168.3.14
* Virtual Machine user/password: vagrant/vagrant
* GitLab webapp running at: http://192.168.3.14:3000/
* GitLab webapp user/password: admin@local.host/5iveL!fe
* MySQL user/password: vagrant/Vagrant
* MySQL root password: nonrandompasswordsaregreattoo
* Xvfb is used as a service and it should be already running, but in case you
  need to restart it manually:

```bash
$ sudo /etc/init.d/xvfb stop
$ sudo /etc/init.d/xvfb start
```


* Install another Ruby: `rbenv install 1.9.3-p448`
* Switch to a different Ruby: `rbenv global 1.9.3-p448`

Updating
---------------

The gitlabhq version is _not_ updated when you rebuild your virtual machine with the following command:

```bash
$ vagrant destroy && vagrant up
```

You must update it yourself by going to the gitlabhq subdirectory in the gitlab-vagrant-vm repo and pulling the latest changes:

```bash
$ cd gitlabhq && git pull --ff origin master
```

A bit of background on why this is needed. When you run 'vagrant up' there is a [checkout action in the recipe](https://github.com/gitlabhq/gitlab-vagrant-vm/blob/master/site-cookbooks/gitlab/recipes/vagrant.rb#L54) that [points to](https://github.com/gitlabhq/gitlab-vagrant-vm/blob/master/site-cookbooks/gitlab/attributes/vagrant.rb#L10) the [gitlabhq repo](https://github.com/gitlabhq/gitlabhq). You won't see any difference when running 'git status' in the gitlab-vagrant-vm repo because gitlabhq/ is in the [.gitignore](https://github.com/gitlabhq/gitlab-vagrant-vm/blob/master/.gitignore). You can update the gitlabhq repo yourself or remove the gitlabhq directory so the repo is checked out again.

Troubleshooting
---------------
### unknown keyword "no_subtree"

Vagrant version 1.3.1 has a bug that prevents guest VM to be mounted.
The error output:

```
[default] Exporting NFS shared folders...
Preparing to edit /etc/exports. Administrator privileges will be required...
nfsd running
exportfs: /etc/exports:1: unknown keyword "no_subtree"

[default] Mounting NFS shared folders...
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

mount -o 'vers=3,udp' 192.168.3.1:'/tmp/gitlab-vagrant-vm' /vagrant
```
[Fix is already provided](https://github.com/mitchellh/vagrant/pull/2156) in the master branch of vagrant repository. Use the version from master or newer release that contains the fix.
