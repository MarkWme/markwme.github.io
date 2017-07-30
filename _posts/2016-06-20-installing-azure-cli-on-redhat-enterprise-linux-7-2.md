---
layout: post
title:  "Installing Azure CLI on RedHat Enterprise Linux 7.2"
date:   2016-06-20 12:11:00 +0100
categories: azure azurecli cli redhat
---
One of the cool things about working with Microsoft Azure, in fact, working with a lot of Microsoft stuff right now, is the cross platform support. A GUI tool, VbScript or PowerShell running on a Windows computer were once essential for administration, but that's no longer the case.

The Azure Cross Platform Command Line Interface provides a tool built using Node.js which can run on Windows, macOS or Linux to provide developers and IT administrators the ability to develop, deploy and manage Microsoft Azure applications on any of those platforms.  And it's open source too - check out the [azure-xplat-cli github repo](https://github.com/Azure/azure-xplat-cli)!

Installation is easy enough. Here's a walkthrough of installing the tools on a fresh, clean instance of RedHat Enterprise Linux 7.2.

### Install Node.js
As the CLI is built with Node.js, you'll need to install that first. There are a couple of choices as to which version of Node you want to deploy and for most production / stable environments you'll likely want the LTS (Long Term Support) version of Node.  At the time of writing, version 4.4.7 is the current LTS version (you can read more about Node.js versions here - https://github.com/nodejs/LTS#lts_schedule).  The following command line will install the repo for the Node.js version 4.x LTS branch:

    curl --silent --location https://rpm.nodesource.com/setup_4.x | sudo bash -

Note the placement of `sudo` in that command line. It's highly likely you'll need root privileges.  Once that's done, we can install Node.js

    sudo yum -y install nodejs

And when that completes, we can check that Node.js is working like this

    node --version

![node-version](/images/2016/07/node-version.png)

### Install Azure Command Line Tools
Now that Node.js is running, we can use the NPM package manager to install the Azure CLI

    sudo npm install azure-cli -g

Again, notice that we use `sudo` to run with root privileges. We also use the `-g` option on the end to ensure the package is installed globally. Once that's done, we can check if the Azure CLI is installed as follows:

    azure --version

![azure-cli-version](/images/2016/07/azure-cli-version.png)

Note that the first time you run the Azure CLI, you will be asked whether you want to opt in to data collection that's used to improve the tools. Select whichever option you prefer, but it you want to change it later you can do so with:

    azure telemetry --disable

or `--enable` as required.

### Login to Azure
And that's it!  You now have the Azure cross platform command line tools installed and running.  So, let's just take this one step further.  Let's login to Azure and confirm we can see our resources as expected. In my case, I'm using a Microsoft corporate account to access Azure and that requires some additional security measures. This might also apply to you if the Azure subscription you're using requires Multi Factor Authentication.

Start with the following command

    azure login

You should then be prompted to open a web browser and navigate to https://aka.ms/devicelogin and enter a unique code that's just been generated for you. So, go do that, open any web browser on any device you have to hand, go to the URL and follow the instructions to enter the device code. Once you've completed that, return to your command line and you should see a message that you're now successfully connected to your Azure subscription.

If you've been using Azure for a while, you'll know that a transition has been underway for some time between the "old" and "new" portals and with that, a transfer from the underlying "Service Management" to "Resource Management" models.  Resource Management (or Azure Resource Management - ARM) is the "modern" way, so lets ensure the CLI is in ARM mode

    azure config mode arm

![azure-config-mode-arm](/images/2016/07/azure-config-mode-arm.png)

And finally, to make sure our connection to Azure looks good, let's get a list of resource groups and make sure that matches what we're expecting to see.

    azure group list

![azure-group-list](/images/2016/07/azure-group-list.png)

Ta da! We're connected to our Azure subscription and we can fully manage our cloud resources. All from a RedHat Enterprise Linux server!

### More Commands
If you want to learn more Azure CLI commands, just type `azure help` and work your way from there. Or, you can take a look at this article in the Azure documentation - https://azure.microsoft.com/en-gb/documentation/articles/azure-cli-arm-commands/
