---
layout: post
title: "Setting Up Your TurtleCoin Public Node"
author: "rndtx"
---

# Setting up and configuring the server

First of all we need a server that will be used to host our public node. You can use your own server or purchase a cheap VPS at DigitalOcean, Google Cloud, Vultr or any others VPS provider. Many cloud providers have a free trial so you can set up a server at no cost.

We'll use an **Ubuntu 18.04 server** for this purpose. The steps will be almost the same for any flavor of Linux. You can use ssh or putty for connecting to your server.

Now we need to add a user as most VPS comes with a root login and it is really not a good idea to run anythis as root (*consider it as a admin with superpowers*). We'll create a user named ***turtlenode*** but you're free to choose any username.

```sh
# adduser turtlenode
```

*Set and confirm the new user's password at the prompt. A strong password is highly recommended! Follow the prompts to set the new user's information. It is fine to accept the defaults to leave all of this information blank.*

Next, run this command to provide admin rights to our server and switch to our new user

```sh
# usermod -aG sudo turtlenode
# su - turtlenode
```

*After running this command, you'llbe able to see the prompt has changed. If you've completed these steps and see a terminal with username as **turtlenode**, congrats! You're already halfway through!*

# Getting TurtleCoin

Now we'll build TurtleCoin on our server. First of all, we need to install all the dependencies.

```sh
sudo apt-get install -y build-essential python-dev gcc g++ git cmake
sudo add-apt-repository universe
sudo apt-get update
sudo apt-get install libboost-all-dev
```

After all the dependencies have been installed, we need to get the [TurtleCoin source code](https://github.com/turtlecoin/turtlecoin) and build using the following commands

```sh
git clone -b master --single-branch https://github.com/turtlecoin/turtlecoin && cd turtlecoin
mkdir build && cd build
cmake .. && make
```

*This will take some time to complete so you can grab a coffee till then. After this step is complete, you'll be able to see new files in the ~/turtlecoin/build/src/ directory. If you don't, then probably the build did not complete successfully and you may want to execute these steps again.*

# Setting up TurtleCoind High-Availability Daemon Wrapper

Now that we have everything ready with us, we need to make sure that our daemon always stays up and on the right chain. For this, we'll be using the [turtlecoind-ha-wrapper](https://github.com/turtlecoin/turtlecoind-ha).

First of all we need to get back to our home directory (~) using

```sh
cd ~
```

Next, we need to install nodejs, a dependency for the wrapper using the following command

```sh
sudo apt-get install nodejs npm
```

Verify if it is installed successfully by running

```sh
nodejs -v
```

This will output the current version of nodejs. If your version is v8.x or higher, you're good to go. Else, you'll need to update.

Next, we need to clone the wrapper and place the daemon to work with the wrapper

```sh
git clone https://github.com/turtlecoin/turtlecoind-ha.git && cd turtlecoind-ha
cp ~/turtlecoin/build/src/TurtleCoind .
```

Now we need to install all the dependencies for the wrapper. Don't worry, we don't need to run every command now as installing dependencies with nodejs is pretty easy. All you need to do is run this single command

```sh
npm install
```

***Note*** : Running *npm install* will automatically fetch and starting syncing from checkpoints for a faster sync.

We now need to set a few parameters to tell the network about our daemon and the fee it charges, for example say 250.00 TRTL per transaction. We need to add these parameters to the file *service.js*. Open up a text editor, in this tutorial i will use nano.

```sh
nano service.js
```

Add the following lines in bold here (*after line number 10*) without missing the commas

```js
var daemon = new TurtleCoind({
    loadCheckpoints: './checkpoints.csv',
    feeAddress: "YOUR TURTLECOIN ADDRES",
    feeAmount: 25000
})
```

Substitute your wallet address and the feeAmount. Please not that the **amount here is in shells**.

1 TRTL = 100 shells, so 25000 shells = 250 TRTL

Save this file with **CTRL + O**, then **CTRL + X**

*As we want our daemon to continue running non-stop 24x7x365, we need something that will help us accomplish that*.

There's a process manager called pm2 wich can be installed with npm, the package manager for ***[nodejs](https://www.npmjs.com/package/pm2)***. The setup is very simple

```sh
sudo npm install -g pm2
```

*Simple, isn't it?*

Now start the daemon with the following command

```sh
pm2 start service.js --name turtlecoind
```

Now if you want to monitor your daemon or want to check the logs, you can do so by

```sh
pm2 monit
pm2 log
```

*Yay! You've successfully set up your own public TurtleCoin node.*

*Congratulations! You are now a TurtleCoin service operator.*

--

If you liked this tutorial, or made some TRTLs running your public node, please consider donating. Thanks!

> TRTLv2Vdfgp3chLbD5NUKMEgkwQjLthtuf3YHLUv6tSATLxDv6VP69N9GH8H6uQF4J1rBr5P1wPfM58sG5NRFXdfPGXoag7fvpz

