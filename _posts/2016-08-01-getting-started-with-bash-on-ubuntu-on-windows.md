---
layout: post
title:  "Getting Started with Bash on Ubuntu on Windows"
date:   2016-08-01 15:47:00 +0100
categories: windows ubunutu bash
---
Bash on Ubuntu on Windows. Now there's a snappy product name! But this somewhat awkward name belongs to one of the best things to happen to Microsoft Windows in a very long time. A Bash shell for Windows.

People like me who like to work with a variety of different technologies often find themselves dual booting Windows and a Linux distribution or using local or cloud based virtual machines running Linux. Why? Because despite the best efforts of some fine folks who do their utmost to get various Linux based open source tools running on Windows, it can sometimes be a real effort to get things working. On quite a few occasions, I've found some cool new tool with a handful of command line entries needed to get it working on Linux, or two pages of pre-requisites and hacks that are needed to get the same thing running on Windows. Or hitting the infamous Windows **[MAX_PATH limitation](https://github.com/nodejs/node-v0.x-archive/issues/6960)**. Often it's just a lot less hassle to do it on a Linux box.

With the Windows 10 Anniversary Update, Microsoft have added Bash support for Windows. Previously, there were ways to get a Linux-like environment up and running on Windows with something like **[Cygwin](https://www.cygwin.com/)** but that still required binaries that were compiled to run on Windows. This is different. This is **native Linux binaries, running on Windows**. There's a lot of detail available on exactly how this has been implemented, **[here](https://blogs.msdn.microsoft.com/commandline/2016/06/02/learn-more-about-bash-on-ubuntu-on-windows-and-the-windows-subsystem-for-linux/)** is a good place to start!

### Installing Bash on Ubuntu on Windows

At a minimum, you need to be running the **Windows 10 Anniversary Update**. This is the one released in August 2016, so if you have an older version of Windows, the first thing to do is upgrade.

Next, Bash is only available to developers and is kept hidden away by default. Windows 10 Anniversary Update adds another great developer feature which is the "Use developer features" window.

![Windows 10 Developer Features](/images/2016/08/buw-devmode-enabled.png)

As an "advanced" user of Windows, there's a ritual I go through on every new install where I go into a dozen or so different menus or windows and change a bunch of settings to make Windows more developer friendly. In the Anniversary Update, a lot of these settings have all been brought together into this one place. To get to this window, just type `developer` into the Start Menu or Cortana's search bar. To install Bash, the item of interest is the radio button that says "Developer Mode".

After you've enabled Developer Mode, there's another feature you need to enable. In the Start Menu or Cortana's search bar, type `features` and look for the "Turn Windows features on or off" option. Then, from the window that appears look for the "Windows Subsystem for Linux(Beta)" entry.

![Windows Subsystem for Linux](/images/2016/08/Windows-Subsystem-for-Linux-1.png)

Once you've enabled this, you'll be prompted to reboot, so go ahead and do that.

When you're back from the reboot, you're ready to get going with the installation of Bash. Click the Start Menu or Cortana's search bar and type `bash`. You should see a bash command like below.

![bash](/images/2016/08/buw-bash-startmenu.png)

Next, you'll see a command window with the following

![bash-install-confirm](/images/2016/08/buw-install-confirm.png)

You're about to install Linux binaries provided by Canonical, so you're asked to confirm that this is indeed what you want to do. If you're ready enter "y" and sit back for a short while. After a bit, you'll be prompted to enter a username and password for the default UNIX user.

![bash-unix-user](/images/2016/08/buw-install-unix-username.png)

As the prompt says, this doesn't have to be the same username that you use to login to Windows. This is a new account that will be used for the Linux subsystem. More details on how this works can be found at **https://aka.ms/wslusers**.

And that's it. You now have Bash on Ubuntu on Windows. Want to prove it? Just try doing something Linux-y! For example, take my earlier blog post on **[installing the Azure CLI on Linux](http://markw.me/installing-azure-cli-on-redhat-enterprise-linux-7-2/)**. Let's install the Azure CLI in Bash.

First, add the repository for Node.js

    curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -

Once that finishes, install Node.js

    sudo apt-get install -y nodejs

And finally lets install the Azure CLI

    sudo npm install azure-cli -g

We'll go through the Azure login process and then list the resource groups we can see in our Azure subscription.

    azure group list

![azure-group-list](/images/2016/08/buw-azurecli-rglist.png)

Or, if we're feeling adventurous, we could try something like using tmux to split the window and run multiple consoles in one!

![tmux](/images/2016/08/buw-tmux.png)

This is a great feature which I think will really help people improve their productivity. Being able to run Linux commands has often been the reason why developers will switch to a Mac. I know I've certainly considered it. But this pretty much negates that argument for the vast majority of developers.
 