[tjmzx]

# Running Lightning Network nodes on macOS 

A recent macOS (laptop or desktop) computer with a wired internet connection can run [bitcoin core] and [lnd] (lightning network daemon) implimentations, both are opensource software. Although macOS is not, running a node and setting it up is also similar to Linux. Performance-wise, most recent desktops tend to be reliable, have enough resources (more than mini-computers like Raspberry Pi) are stable, easy to configure and maintain.

### Which LN implimentation?
This guide describes steps for installing and configuring [lightning.engineering]'s implimentation, [lnd]. Different implimentations may have different advantages. If you are going to run a node as a serious commitment, you may do well to research which implimentation may work better for your needs. Compare the [lnd repo on Github] with [Elements.Project]'s [CoreLightning repo on Github] as a start. Both are widely used, maturing, and have great documentation and support. *The setup has been tested on MacbookPro Intel and ARM (Silicon) architectures.

| Hardware | Minimum Recommended Specification |
| ------ | ------ |
| Model | 13-inch, 2020 (Ethernet + 1 USB / Thunderbolt port |
| Processor | 2 GHz Quad-Core Intel Core i5 |
| Memory | 16 GB 3733 MHz LPDDR4X |
| Storage | 1-2TB SSD (internal or external storage volume) |

### Software / OS version
To install the required dependencies, both [Xcode Commandline tools] and [Homebrew] are required, [see Homebrew installation notes]. This means a 64-bit Intel or ARM (Silicon) CPU, on Catalina (10.15) or above is needed, but consider that more recent OS releases have the benefit of up-to-date tools and software dependencies. This installation has been tested on macOS 12.0 - 12.4 (Monterey) to the latest update

### Hardware
- Apple mac (laptop, mini or tower)
- a wired connection (CAT6 ethernet) to a standard router
- Minimum 1TB (preferably 2TB for future-proofing) external SSD

## Preparation

### OS Software Update 
Either navigate via menubar `` ▶︎ `System Preferences` ▶︎ `Software Update` or open `/Applications/Utilities/Terminal.app` then run:
```sh 
softwareupdate -l
```
This should check for `Xcode Commandline tools` which the Homebrew installation includes, or it can optionally be installed with:
```sh
xcode-select --install
```

### Finder: Reveal/hide dot-files
If you navigate using gui on macOS, you may need to  hidden files in the finder. The key commmand to temporarily toggle Hidden (dot files') visibility:
``` 
keys: cmd + shift + [.] T
```

Or to permanently toggle Hidden Files' visibility ..
```sh 
defaults write com.apple.Finder AppleShowAllFiles -bool YES; killall -HUP Finder
```
to revert (if needed) ..
```sh 
defaults write com.apple.Finder AppleShowAllFiles -bool NO; killall -HUP Finder
```
## Package managers
Either [Homebrew] 🍺 or [MacPorts] should work fine. For this guide we will install Homebrew. 

### Shell
Run the first installation from [bash] (the standard shell on macOS Terminal is now zsh)
To check you are using bash, from Terminal.app run ..

```sh 
echo $SHELL
```
The output should be ..
```
/bin/bash
```
however, if the output is
```
/bin/zsh
```
use the following command to list all available shells (you need to make sure you have `/bin/bash`
```sh
cat /etc/shells
```
To set bash in terminal.app: `Terminal ▶︎ preferences ▶︎ Shells ▶︎ open with ▶︎ command` and insert `/bin/bash` in the `command` field. You can then reselect `default` to change it back to zsh after installing Homebrew. 

## Install Homebrew

To install homebrew using curl ..

```sh 
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Update Homebrew and upgrade packages
```sh
brew update && upgrade && cleanup
```
Check for and uninstall conflicting dependency (qt)
```sh
brew uninstall qt
```
## Install bitcoin core's dependiencies 
```sh
brew install berkeley-db@4 qt@5 qrencode miniupnpc libnatpmp zeromq python pip3 ds_store mac_alias
```

## [Build bitcoin core osx]
bitcoind is all that is required to run a node. The settings below will install both bitcoin CLI (bitcoind) and the GUI (bitcoin core)
The wallet and legacy berkeley database are also configured so that bitcoin core can be used fully as a standalone wallet.

If you wish to streamline the installation, you could save both diskspace and time building it by configuring it without (lnd does not use bitcoin core's wallet and creates its own wallet instead.)

make a directory and move to it
```sh 
mkdir ~/bitcoin && cd ~/bitcoin
```
clone the gh repo
```sh 
git clone bitcoin
```
move into repo (prompt should change to 'bitcoin git:(master)')
```sh 
cd bitcoin/bitcoin
```
Check releases for latest stable release tag at https://github.com/bitcoin/bitcoin/releases/ (v23.0 at time of writing)
```sh
git checkout v23.0 
```
Configure with gui, wallet and berkeley db
```sh
./autogen.sh ./configure --with-gui=yes
```
make (-j* = n of cores + 1 utilized)
```sh
make -j5   
```
### tor (optional)
Installing [tor] is not necessary but if you want an easy way to start running without exposing your IP, enabling it is a good way to start. There are a few minor differences in configuring its depending, depending on your architecture. To install `tor` and `torsocks` for ..
```sh
brew install tor torsocks
```
Intel. Create `torrc` configuration file
```sh
sudo nano /usr/local/etc/tor/torrc
```
Silicon (M1/M2) Create `torrc` configuration file
```sh
sudo nano /opt/homebrew/etc/tor/torrc
```
When you have created thr `torrc` file at the corret path, add these lines in `torrc` 
```
ControlPort 9051
CookieAuthentication 1
CookieAuthFileGroupReadable 1
Log notice stdout
SOCKSPort 9050
```
## Configuring bitcoin

The `bitcoin.conf` file controls various functions of bitcoin's operation (both bitcoind and bitcoin core.) 
These are suggested settings to interact with lightning (without using a bitcoin wallet.)

Make a data directory (BTC) * You can name it anything
```
mkdir /Volumes/MySSD/BTC
```
Make a `blocks` subdirectory in the data directory (BTC/blocks)
```
mkdir /Volumes/MySSD/BTC/blocks
```
Create `bitcoin.conf` file and write the settings as below 
```
sudo nano /Volumes/MySSD/BTC/bitcoin.conf
```
`bitcoin.conf` _* Don't forget to change `data directory` `blocks` and `.cookie` to your directories' path_
```
# Set the data directory to the SSD (mySSD)
datadir=/Volumes/MySSD/BTC

# Set the blocks directory to the SSD (mySSD)
blocksdir=/Volumes/MySSD/BTC/blocks

# Set rpc cookie file
rpccookiefile=/Volumes/MySSD/BTC/.cookie

# Set the number of megabytes of RAM to use, set to like 50% of available memory
dbcache=3000

# Debugging
debug=mempool
debug=rpc
debug=tor

# Turn off the wallet (no need for a core wallet)
disablewallet=1

# Don't listen for peers (only outgoing peer connections)
listen=0

# Constrain the mempool to the number of megabytes needed:
maxmempool=100

# Limit uploading to peers
maxuploadtarget=1000

# Turn off serving SPV nodes
nopeerbloomfilters=1
peerbloomfilters=0

# Don't accept deprecated multi-sig style
permitbaremultisig=0

# Turn on the RPC server (for remote procedure calls)
server=1

# Reduce the log file size on restarts
shrinkdebuglog=1

# Turn on transaction lookup index
txindex=1

# Turn on ZMQ publishing (lightning network utilizes ZMQ for communication with bitcoind)
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28333
```
### *Note on Authenitication of RPC (Remote procedure call)

The potential for malicious code/hacks (attack-vectors) are minimized by utilizing a temporary authentication procedure 'cookie' which is renewed when bitcoind starts. There is another authentication method which utilizes python to create a user/pass-key. The script can be found within bitcoin-core's repo. To use this, first check you have installed `python3` with ```brew install python3```  then   `cd` into the `/share/rpcauth` directory and run:
```
./rpcauth.py your-desired-username
```
This will output credentials which you can then use as a _permanent_ authentication method. However specifying 'cookiefile' is recommended. 
*Further notes and research about RPC authentication security can be found here: [Kixunil]

## go

In order to run lnd, the [go language] must be installed. This can be done through [installing from the go website] or simply using Homebrew. Package managers make it trivial to update software. You can choose whichever way you geel more comfortable with. The latest install can be fetched via Homebrew with:

```sh
brew install go
```
or to [install from the go website] find the correct download for your acrhitecture Intel / ARM and follow the [instructions]

You need to make sure you have go in your path. You can use an editor such as `nano` or any text editor to edit.
If your Terminal shell is bash:
```sh
nano ~/.bashrc
```
If your Terminal shell is zsh:
```sh
nano ~/.zshrc
```
and add the following lines that are the default install locations for go
```sh
export GOPATH=~/go
export PATH=$PATH:$GOPATH/bin
```
## lnd

Below are the generic installation notes taken from lnd's github repo. The only things to be aware of is the location of the designated default folders used for lnd on macos, which are:


This is the folder which stores the database and all the relevant config files. When installing, clone the github repo to a folder where you can access and run updates in the future (somewhere in the /Users/Yourusername/ folder. This is the equivilant to `/home` in Linux systems. so first cd into this and create a subfolder if necessary.

```sh
mkdir /Users/Yourusername/lightning_code
cd /Users/Yourusername/lightning_code
```
Then clone

```sh
git clone https://github.com/lightningnetwork/lnd
```
```sh
cd lnd
```
Check for the [current stable release version] and then `git checkout` + v0.xx.x-beta

```sh
git checkout v0.15.0-beta
```

Then make the release

```sh
make install
```

### Configure lnd.conf (lnd's configuration file)


## gpg (gnu-privacy-guard for auto unlock)

## LaunchAgents and LaunchDaemons


[//]: # (These are reference links.)

   [tjmzx]: <https://zebitz.uk/index.html>

   [apple user account]: <https://appleid.apple.com/>
   [bitcoin core]: <https://github.com/bitcoin/bitcoin>
   [LN]: <https://lightning.network/>
   [lnd]: <https://github.com/lightningnetwork/lnd>
   [lnd repo on Github]: <https://github.com/lnd>
   [current stable release]: <https://github.com/lightningnetwork/lnd/releases>
   [lightning.engineering]: <https://docs.lightning.engineering/lightning-network-tools/lnd/wallet>
   [Elements.Project]: <https://elementsproject.org/>
   [CoreLightning repo on Github]: <https://github.com/ElementsProject/lightning>
   [Homebrew]: <https://brew.sh>
   [MacPorts]: <https://www.macports.org/>
   [Kixunil]: <https://github.com/Kixunil/security_writings/blob/master/cookie_files.md>
   [tor]: <https://torproject.org/>
   [Build bitcoin core osx]: <https://github.com/bitcoin/bitcoin/blob/master/doc/build-osx.md>
   [see Homebrew installation note]: <https://docs.brew.sh/Installation>
   [rust]: <https://rust-lang.org>
   [install rust]: <https://doc.rust-lang.org/book/ch01-01-installation.html>
   [go language]: <https://go.dev/>
   [installing from the go website]: <https://go.dev/dl/>
   [instructions]: <https://go.dev/doc/install>
   [Xcode commandline tools]: <https://developer.apple.com/xcode/resources/>

