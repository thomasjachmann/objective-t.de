---
layout: post
title:  "Using Gitosis on DreamHost"
date:   2009-05-11 16:08:00
categories: tech
---

In order to use gitosis on your DreamHost account, you'll need to create a dedicated user, install gitosis and start configuring gitosis/pushing to your repositories.


## Create User

It is best to use this user for gitosis only, since it modifies ```.ssh/authorized_keys``` and you might not be able to access the account using ```ssh``` when using pubkey authentication since it restricts shell access to ```gitosis-serve``` only. So if you need to do something on the account later and you're blocked to log in, you still can ```ssh``` into the system using another user and ```su``` into your gitosis user. Create a user using the [web panel](https://panel.dreamhost.com/index.cgi?tree=users.users&amp;), then login in using that user.


## Prepare site-packages Installation

Gitosis uses python's ```setuptools``` for installation. This will install to the system's ```site-packages``` directory (usually ```/usr/lib/python2.4/site-packages/```). In a shared hosting environment, you don't have write access to this directory. First, create a local directory structure in ```~/opt/``` where everything will be installed to and then modify some environment variables for these directories to be used.

```bash
# create directory structure
mkdir -p ~/opt/bin ~/opt/lib/python2.4/site-packages

# adjust environment variables
echo "export PATH=$HOME/opt/bin:$PATH" >> ~/.bashrc
echo "export PYTHONPATH=$HOME/opt/lib/python2.4/site-packages" >> ~/.bashrc

# load changes
source ~/.bashrc
```


## Install Git

If the version of git (1.5.6.5 at time of writing) is sufficient for you, you don't have to install a current git version. If you need to, see the references below for instructions on how to do it.


## Install Gitosis

While installing gitosis, you need to tell the installer to install it to your local directory structure.

In order to setup gitosis, you'll need your personal pubkey (in this example, it is assumed that you transferred the file ```id_dsa.pub``` - or however your pubkey's named - to the gitosis user's home folder). This will be used to authorize the initial admin user access.

In order for the administrative functions of gitosis to function, the post-update hook needs to be executable. This will export ```.gitosis.conf``` and adjust ```.ssh/authorized_keys``` so that the configured users have access to the system.

```bash
# download gitosis
git clone git://eagain.net/gitosis.git

# install gitosis
cd gitosis
python setup.py install --prefix=$HOME/opt

# initial setup
gitosis-init < ~/id_dsa.pub

# adjust access rights
chmod u+x ~/repositories/gitosis-admin.git/hooks/post-update

# remove installation files
cd ..
rm -rf gitosis
```

This is all you've got to do on the server.


## Configure Gitosis

```bash
# download gitosis' configuration repository
git clone <youruser>@<yourserver>:gitosis-admin.git
cd gitosis-admin

# add pubkeys into keydir and edit gitosis.conf
# for example add your pubkey and add it to a new group devs
# that has write access to a new project repository
cp ~/.ssh/id_dsa.pub keydir/thomas.pub
echo "[group devs]
writable = new_project
members = thomas" >> gitosis.conf

# upload changes to server
git add .
git commit -m 'configuration changes'
git push
```

Note that the remote repository doesn't exist yet. Just push to it in the next step and it will be created automatically.


## Create Repository And Push To Server

```bash
# create project
mkdir new_project
cd new_project

# init local git repository
git init

# do something, add and commit

# add remote repository and add tracking
git remote add origin <youruser>@<yourserver>:new_project.git
git config branch.master.remote origin
git config branch.master.merge refs/heads/master
# next line needed for git > 1.6.3 to avoid warning messages
git config push.default current

# push to remote repository
git push origin master
```

From now on, you can use git pull/git push since tracking has been established.


## References

* http://scie.nti.st/2007/11/14/hosting-git-repositories-the-easy-and-secure-way
* http://openmonkey.com/blog/2009/02/25/installing-gitosis-on-dreamhost
* http://thelucid.com/2008/12/02/git-setting-up-a-remote-repository-and-doing-an-initial-push
* http://pivotallabs.com/users/alex/blog/articles/883-git-config-push-default-matching
