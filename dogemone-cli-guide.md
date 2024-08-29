# Reference Client overview

## General overview

Dogemone works through 2 separate binary files operated through command line:

1. **dogemoned** – daemon to synchronize the blockchain and mine dmecoins.
2. **simplewallet** – wallet to receive and send funds.

Make sure to place these files in one separate folder.


## Install Dogemone

1. Unzip the archive to one separate folder. Dogemone will generate
additional data files and log files in the same directory.
The blockchain data is stored in `~/.dogemone` on Linux/Unix systems,
`C:\Users\%USERNAME%\AppData\Roaming\dogemone` on Windows or in
`~/Library/Application support/` on Mac OS.

## Launch Dogemone

1. Open **dogemoned**

2. Wait until **dogemoned** is synchronized. This may take a while. You will be
notified with several green "SYNCHRONIZED OK" messages.

3. Open **simplewallet**.


## Daemon: synchronization and mining

After **dogemoned** is launched you will have to wait until it is synchronized with
the network. You will be notified with several green "SYNCHRONIZED OK" messages.
Here are the most important daemon commands:
* `start_mining <SECRET_SPEND_KEY> <SECRET_VIEW_KEY> [threads=1]` Start mining in several threads to a given wallet
* `stop_mining`					Stop mining
* `show_hr`						Show current mining hashrate
* `hide_hr`						Stop showing current mining hashrate
* `help`						Show all daemon commands
* `exit`						Exit dogemoned

Please refer to [Mining guide](mining.md) for more info.

## Wallet: send and receive payments

Simplewallet can only be used after the daemon is launched and synchronized.
Simlewallet automatically synchronizes with dogemoned. Here are the most
important wallet commands:

* `address`						Show your wallet address
* `balance`						Show current wallet balance
* `transfer <mixin_count> <address> <amount>`	Send money to `<address>` with a mixing amount of `<mixin_count>`
* `start_mining <threads>`		Start mining in daemon with several threads to the current wallet address
* `stop_mining`					Stop mining in daemon
* `help`						Show all wallet commands
* `exit`						Exit simplewallet
