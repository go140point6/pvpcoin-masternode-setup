# PvPCoin Masternode Setup Guide

## Introduction
This guide is for a single masternode, (tested) on an Ubuntu 18.04 64bit server (VPS) running headless (no GUI desktop) and controlled from the wallet on your local computer (Control wallet). The wallet on the the VPS will be referred to as the Remote wallet.
You will need your server details for progressing through this guide.

Note: There are many different ways to set up masternodes, but this method is considered cold-staking (remote wallet has zero balance because the pledge, in this case 888 pvpcoin, stays on the control wallet) as opposed to hot-staking (remote wallet has the 888 pvpcoin).  Cold-staking a masternode is generally considered more secure. 

<b>Proceed at your own risk! These instructions worked for me as written, but you decide how to proceed. Any questions, you can call me out on discord (@go140point6#7708). If you find me on discord (https://discord.gg/RNxbQcE), I will *never* ask for your private keys, passwords or other sensitive information. If someone calling themselves go140point6 does, it isn't me, and you should block them.</b>

First the basic requirements:
- 888 pvpcoin (you need a bit more, there is a small transaction cost obviously).  This is in your control wallet.
- A main computer (Your everyday computer or even a Pi) – This will run the control wallet, hold your collateral 888 pvpcoin and can be turned on and off without affecting the masternode.
- For a static build x86_64 linux version of the wallet (-qt, -cli and daemon) or a static Pi build (-cli and daemon only), see my other repo (https://github.com/go140point6/pvpcoin-build/releases). For a Windows version, see the "official" repo (https://github.com/MPAds/PvPCoin/releases).
  - pick a suitable version for your OS
- Masternode Server (VPS or local – The computer that will be on 24/7... if local, why not make it a Pi?)  
  - I use a VPS at [vultr.com] (https://www.vultr.com/?ref=8246948-4F), which costs $5/month (a $3.50/month VPS would work too). If that link is expired, please try [vultr.com] (https://www.vultr.com/?ref=8246947).  The first link will get you $50 for the first month to test out the service (more than enough to cover a masternode).  Not sure if the second link gives you anything, I was able to use the first link when I first signed up.
- A unique IP address for your VPS / Remote wallet
  - You will get those from your hosting party.

(For security reasons, you’re are going to need a different IP for each masternode you plan to host)

The basic reasoning for these requirements is that you get to keep your pvpcoin in your local wallet and host your masternode remotely, securely.

I am writing these instuctions using playervsplayercoin-cli, not the -qt (GUI) wallet.  These commands will work in the -qt wallet if you want, just go to the debug console (Tools > Debug console) and type them in just as if you were at the command line.  I completed these steps on Lubuntu 19.04 using the -cli wallet.

Since you probably have the 888 pvpcoin needed to setup and run a masternode, it is assumed you already have a running control wallet on either Linux or Windows.  Remember, if you have a running -qt, just use these commands in the Debug console, except where noted.

A note about encryption: It is ALWAYS a good idea to encrypt a wallet that has funds.  Please be sure to encrypt your wallet and keep the password in (multiple) safe locations.  No password, no access to funds.  And if you have backup copies of your wallet before it was encyrpted, get rid of them!  Someone with access to those unecrypted wallets wouldn't need your passphrase.  You decide how to proceed, this document assumes you have encrypted your control wallet... no need to encrypt your remote wallet (masternode), as it contains no funds.

Hard to see, but there is a space in front of the following command.  This is very important because your shell likely keeps a history of your commands.  Type ```history``` to see what I'm talking about.  This is a very useful tool but doesn't work well with cleartext passwords.  So place a space before your command and it will not be recorded in history.  If you forget, simply type ```history``` and find the line you want to remove, then ```history -d <linenumber>``` to remove that line.
```
 ./playervsplayercoin-cli encryptwallet <mylongrandompassphrase>
```
another note... special characters are allowed in the passphrase, but must be escaped with a backslash (\\).  I recommend you avoid the exlaimation point (!), it is a pain to use on the -cli and I always have issues trying to escape it.  Here's an example using a strong password with escaped characters:
```
./playervsplayercoin-cli encryptwallet kj\$d4f9\#kdw8
```
the password you see above is actually kj$d4f9#kdw8 but you must escape the special characters in the Linux console.  When unlocking this wallet on -qt, you won't need (and shouldn't try) to escape them.


## Configuration
1. Using the control wallet, type the following command:
```
./playervsplayercoin-cli masternode genkey
```
This will be the masternode’s privkey. Save this output, we’ll use this later…


2. Using the control wallet still, enter the following command:
```
./playervsplayercoin-cli getaccountaddress chooseAnyNameForYourMasternode
```
This is the address that will hold your 888 pvpcoin.  Give it a name (i.e. MN1).  I don't recommend you use this address for anything else, the 888 pvpcoin will be locked.  Your masternode rewards will be sent to this address as well, but those funds should be available to you once confirmed like any other transaction.


3. Still in the control wallet, send pvpcoin to the address you generated in step 2 (Be 100% sure that you entered the address correctly. Make sure this is exactly 888 pvpcoin; No more, no less.).  STOP! If using the -qt wallet, just use the Send tab like you normally would, no reason to do this from the debug console.  If you are at all uncomfortable with this step on the -cli, use the -qt wallet.  If you are on a system that doesn't have a working -qt (i.e. Pi), then move your wallet to a system that does to complete these steps.  Once the control wallet is setup, simply move it back.

You DID encrypt this wallet, didn't you?  Don't forget to escape any special characters (see the top).  The number after the passphrase is the time in seconds that the wallet remains unlocked, you only need it unlocked long enough to initiate your transaction.  And again, note the space in front of the command unlocking your wallet!
```
 ./playervsplayercoin-cli walletpassphrase <mylongrandompassphrase> 120
./playervsplayercoin-cli sendtoaddress <pvpcoinaddress> 888
```
 - Be absolutely 100% sure that this is copied correctly. And then check it again. I can not help you, if you send 888 pvpcoin to an incorrect address.
 
 Note that transaction id that gets produced.  You'll need it to check your number of confirmations with the following command:
 ```
 ./playervsplayercoin-cli gettransactions <reallylongtransactionid>
 ```


4. Time for a beer (or your drink of choice).  You need 777 confirmations and that will take awhile.  Go play outside.


5. Still in the control wallet, enter the command into the console:
```
./playervsplayercoin-cli masternode outputs
```
 - This gets the proof of transaction of sending 888 pvpcoin and only works after you get 777 confirmations.


6. Still on the control wallet, find your masternode configuration file (masternode.conf).  On Linux this is in the .playervsplayercoin directory (typically in your home folder).  On Windows, it's typically in your home folder in the hidden AppData/Roaming folder (somewhere, I didn't test any of this on Windows).
  - On Linux you can type the following in your terminal (the nano editor is easy to use, but substitute your editor of choice... I personally use VIM to do my editing).
```
nano ~/.playervsplayercoin/masternode.conf
```
Add the following line to the configuration file:
```
<Name of Masternode(Use the name you entered earlier for simplicity)> <Unique IP address>:4568 <The result of Step 1> <Result of Step 5> <The number after the long line in Step 5>
```
Substitute it with your own values and without the “<>”s  
```
Example: MN1 31.14.135.27:4568 892WPpkqbr7sr6Si4fdsfssjjapuFzAXwETCrpPJubnrmU6aKzh c8f4965ea57a68d0e6dd384324dfd28cfbe0c801015b973e7331db8ce018716999 0
```
 - to exit nano press
 ```
 CTRL + X
 ```
 then
 ```
 Y
 ```
 then
 ```
 ENTER
 ```


7. Restart your control wallet.
    - On windows just close the QT wallet and re-open it
    - On linux you can close the QT (if your using it) or restart the daemon as follows:  
    Go to the directory where the daemon and the -cli are, and stop pvpcoin with:
    ```
    ./playervsplayercoin-cli stop
    ```
    Now start it up again, note the different binary:
    ```
    ./playervsplayercoind -daemon
    ```
    
8. Let's give it a few minutes to come back up and get connections.  When the following command give you a couple steady connections, you should be good to proceed:
```
./playervsplayercoin-cli getinfo
```


## VPS Remote wallet install
Connect to your vps or the local server that will be on 24/7. This is already assumed to be running with your OS of choice, and is out of scope of this document. 
 
1. Install the latest version of the PvPCoin wallet onto your masternode. The lastest "official" version can be found here: [PvPCoin releases](https://github.com/MPAds/PvPCoin/releases) but as of this writing those Linux binaries require many dependencies to get running (Windows may be OK, I didn't check).  I have compiled the Linux and Arm (Pi) binaries statically, so they run on a clean system (tested Ubuntu 18.04, Lubuntu 19.04 and Rasbian Buster). You can download my binaries or follow my instructions to compile your own copies here: [pvpcoin-releases] (https://github.com/go140point6/pvpcoin-build/releases).
 - Go to your home directory:
 ```
 cd ~ && mkdir -p ~/src && cd ~/src
 ```
 - From your home directory, download the latest version from the GitHub repository of your choice. To get the latest version copy the link for linux from step 1.  
 ** Note this is an example url, always make sure you have the latest version! **
 ```
 wget https://github.com/go140point6/pvpcoin-build/releases/download/v0.12.3.3/pvpcoin-0.12.3.3-x86_64-linux-gnu.tar.gz
 ```
 - Uncompress and extract:
 ```
 tar zxvf pvpcoin-0.12.3.3-x86_64-linux-gnu.tar.gz -C ~/
 ```
 - Go to the location it uncompressed to:
 ```
 cd ~/pvpcoin-0.12.3.3/bin/
```
 - Note: If this is the first time running the wallet in the VPS, you’ll need to start the wallet up for a few seconds:
 ```
 ./playervsplayercoind
 ```
    this will create the config files in your ~/.playervsplayercoin data directory
 after a few seconds:
    ```
    ./playervsplayercoin-cli stop
    ```
    to exit / stop the wallet then continue to the next step

2. Still on remote server (masternode), find the PvPCoin data directory (on Linux: ~/.playervsplayercoin)
```
cd ~/.playervsplayercoin
```


3. Open playervsplayer.conf (again, using nano as an example, use your editor of choice) by typing:
```
nano playervsplayercoin.conf
```
 - Add the following lines to the file.
 ```
rpcuser=long random username
rpcpassword=longer random password
rpcallowip=127.0.0.1
server=1
daemon=1
logtimestamps=1
maxconnections=256
masternode=1
externalip=<your unique public ip address>
masternodeprivkey=<what you generated using the masternode genkey command at the very beginning>
addnode=140.186.30.102:4568
addnode=176.254.153.206:4568
addnode=162.237.70.56:56816
addnode=162.237.70.56:58343
addnode=162.237.70.56:59114
addnode=136.144.171.201:4568
addnode=22.137.226.5:4568
addnode=136.144.171.201:4568
addnode=155.138.162.120:4568
addnode=207.246.108.36:4568
addnode=3.215.162.101:4568
addnode=3.230.179.87:4568
addnode=50.245.85.76:4568
addnode=149.28.160.187:4568
addnode=50.245.85.73:4568
addnode=50.245.85.74:4568
addnode=50.245.85.77:4568
addnode=207.154.242.254:4568
```
Make sure to replace rpcuser and rpcpassword with your own and make them long and random.  You will never need to actually use this username/password combination.  Also be sure to replace <your unique public ip address> with your VPS ip address and use your generated masternode private key.

 - to exit nano press
 ```
 CTRL + X
 ```
 then
 ```
 Y
 ```
 then
 ```
 ENTER
 ```
 

## Start your masternode
Now you are ready to start things in this order (this assumes your control wallet is running and current with the blockchain, if not, start it and wait for it to sync):

1. On the control wallet, you DID encrypt this wallet, didn't you?  Don't forget to escape any special characters (see the top).  The number after the passphrase is the time in seconds that the wallet remains unlocked, you only need it unlocked long enough to start up the masternode.

Again, note the space in front of the command to unlock the wallet.  This keeps your passphrase from being recorded in the shell's history:
```
 ./playervsplayercoin-cli walletpassphrase <mylongrandompassphrase> 120
./playervsplayercoin-cli masternode start-alias <mymnalias>
```
where \<mymnalias\> is the name of your masternode alias (without brackets).  In our example above, this was MN1.

The following should appear:
```
{
  "alias": "MN1",
  "result": "successful"
}
```
  if you get an error saying that the alias is not found then make sure you typed it correctly by opening the masternode configuration file and checking. If this is correct try restarting your control wallet.


2. Back on the remote server (masternode), start the wallet:
```
./playervsplayercoind
```

As this is your first time starting the wallet to let it run, you'll need to download the blockchain, which could take some time.  Use the handy 'tail' command and then go get another beer:
```
tail -f ~/.playervsplayercoin/debug.log
```
You'll see lots of stuff flashing by, let it run.  You are looking for it to slow a bit once caught up to the blockchain.  Hit ```ctrl-x``` to exit.


3. Still on the remote server (masternode):
```
./playervsplayercoin-cli masternode status
```
The following similar output should appear:
```
{
  "outpoint": "ce4e14920e248b45120d7aaac7693e4582ee26ec6f2fae72002ee281bb5426bd-1",
  "service": "207.246.108.36:4568",
  "payee": "PRa9qEkTDvbJCYAdeXfxazmt3H827A4hLm",
  "status": "Masternode successfully started"
}
```

 
4. Now, run the following and you should find your ip address in the list... also you should see ```"status": "PRE-ENABLED"```.  You want ```ENABLED``` but you need to restart your contol wallet first and let it sync up again:
```
 ./playervsplayercoin-cli masternode list
```
The following output should appear (truncated to just my masternode):
```
  "ce4e14920e248b45120d7aaac7693e4582ee26ec6f2fae72002ee281bb5426bd-1": {
    "address": "207.246.108.36:4568",
    "payee": "PRa9qEkTDvbJCYAdeXfxazmt3H827A4hLm",
    "status": "PRE-ENABLED",
    "protocol": 70210,
    "daemonversion": "0.12.3.3",
    "sentinelversion": "Unknown",
    "sentinelstate": "expired",
    "lastseen": 1568136852,
    "activeseconds": 674,
    "lastpaidtime": 0,
    "lastpaidblock": 0
```

5. Back over on the control wallet, restart and let it sync:
```
./playervsplayercoin-cli -stop
./playervsplayercoind -daemon
```


6. From either control wallet or remote wallet, you can check to see if your masternode is up and running as ```ENABLED```:
```
./playervsplayercoin-cli masternode list
```
You are looking for:
```
  "ce4e14920e248b45120d7aaac7693e4582ee26ec6f2fae72002ee281bb5426bd-1": {
    "address": "207.246.108.36:4568",
    "payee": "PRa9qEkTDvbJCYAdeXfxazmt3H827A4hLm",
    "status": "ENABLED",
    "protocol": 70210,
    "daemonversion": "0.12.3.3",
    "sentinelversion": "Unknown",
    "sentinelstate": "expired",
    "lastseen": 1568137452,
    "activeseconds": 1274,
    "lastpaidtime": 0,
    "lastpaidblock": 0
```
It takes some time for your masternode to sync with the other masternodes, so if it says PRE-ENABLED at first, don't panic.  Just wait a few minutes (you have another beer, right?) and run the command again.

Congratulations! You have successfully created your cold-staking masternode!  Your pvpcoins are safely stored in your control wallet!

I believe you issue the following command to see that transaction as being unspendable (but not certain about this):
```
./playervsplayercoin-cli listlockunspent
```


## Tearing down a Masternode
1. How do I stop running MN1 on my VPS host and delete MN1 from my PvPCoin Core wallet?
 > 1) From the remote wallet (masternode), stop the wallet.  
 ```
 ./playervsplayercoin-cli stop
 ```
 > 2) Then from your control wallet, edit your masternode.conf, delete the MN1 masternode line entry.  
 > 3) Restart the control wallet.  
 > 4) your 888 will now be unlocked.  
2. How do I get the 888 back that I’ve sent to my MN1 address at the beginning of the MN1 setup?
 > You don’t need to “get that back” as it is already in your wallet.
Being in a different address is not an issue as that’s also your address.
3. Can I use this 888 normally then again, and sell it or give it away?
 > yes!
