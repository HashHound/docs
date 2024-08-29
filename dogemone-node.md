# How to run Dogemone node as Linux service

On this page you will find description how to run dogemoned with JSON PRC as Linux service that will restart itself if daemon crashes for some reason. This example is for Ubuntu server 18.04 x64, but it can be applied to any of the linux versions with small changes.

## Create Linux service to start dogemoned

1. To start service we will use user _dogemone_, so lets create it, manage permissions and login:

```
useradd -m -s /bin/bash -G adm,systemd-journal,sudo dogemone && passwd dogemone
groupadd dogemone
usermod -a -G dogemone dogemone
su dogemone
```

2. Create directory _/DOGEMONE/_ in home directory of _dogemone_ user:

```
cd ~
mkdir DOGEMONE
cd DOGEMONE
```

3. Download latest Linux version of Dogemone and unpack it:

This one is for Linux 18.04, if you have other version, find a corresponding release at https://github.com/HashHound/Dogemone/releases/

```
wget https://github.com/HashHound/Dogemone/releases/download/v.1.7.6/Dogemone-cli-ubuntu18.04-v.1.7.6.tar.gz
tar -xvzf Dogemone-cli-ubuntu18.04-v.1.7.6.tar.gz
rm Dogemone-cli-ubuntu18.04-v.1.7.6.tar.gz
```

4. Create log file and add permision to write it:

```
sudo touch /var/log/dogemoned
sudo chgrp -R dogemone /var/log/dogemoned
sudo chmod -R 770 /var/log/dogemoned
```

5. Optionally you can pre-download blockchain bootstrap to speed-up process:

```
cd DOGEMONE
mkdir .dogemone
cd .dogemone
wget https://bootstrap.dogemone.online/blockchain-$(date "+%Y-%m-%d").tar.gz
tar -xvzf blockchain-$(date "+%Y-%m-%d").tar.gz
rm -f blockchain-$(date "+%Y-%m-%d").tar.gz
```

Exit to `root`
```
exit
```

6. Add Dogemone to firewall:

```
apt-get install ufw -y
ufw default allow outgoing
ufw default deny incoming
ufw allow ssh/tcp
ufw limit ssh/tcp
ufw allow http/tcp
ufw allow https/tcp
ufw allow 53000/tcp comment "DOGEMONE"
ufw logging on
ufw -f enable
systemctl enable ufw
ufw status
```

Retun to `dogemone`
```
su dogemone
cd ~
```

7. Lets check if everything is ok. Try to run daemon with _dogemone_ user permission and wait for SYNCHRONIZED OK.

o not forget to change address to your wallet!
```
sudo -u dogemone DOGEMONE/dogemoned --data-dir=DOGEMONE/.dogemone --log-file=/var/log/dogemoned --restricted-rpc --enable-cors=* --enable-blockchain-indexes --rpc-bind-ip=0.0.0.0 --rpc-bind-port=53000 --fee-address=KixW1oDMYMyabY6CdwS13fQuUwmUeXTPThF9s9JfDggbXSQGnZXkrP9LvcJtV9x7qb2pLsSobkXWXCrPsGGeC1V6VPBhPva --fee-amount=0.1
```
After that, stop it via entering `exit` inside daemon session.

If you facing errors, you could run Dogemone node with debug:
```
sudo -u dogemone DOGEMONE/dogemoned --data-dir=DOGEMONE/.dogemone --log-file=/var/log/dogemoned --restricted-rpc --enable-cors=*  --enable-blockchain-indexes --rpc-bind-ip=0.0.0.0 --rpc-bind-port=53000 --log-level=4 --fee-address=KixW1oDMYMyabY6CdwS13fQuUwmUeXTPThF9s9JfDggbXSQGnZXkrP9LvcJtV9x7qb2pLsSobkXWXCrPsGGeC1V6VPBhPva --fee-amount=0.1
```

Exit to `root`
```
exit
```

8. To autostart _dogemoned_ daemon, we need to create service file in _/etc/systemd/system_

```
sudo nano /etc/systemd/system/dogemoned.service
```
with such content:
```
[Unit]
Description=dogemoned
Documentation=http://dogemone.online
After=syslog.target

[Service]
User=dogemone
Restart=always
RestartSec=10
ExecStart=/home/dogemone/DOGEMONE/dogemoned --data-dir=/home/dogemone/DOGEMONE/.dogemone --log-file=/var/log/dogemoned --restricted-rpc --enable-cors=*  --enable-blockchain-indexes --rpc-bind-ip=0.0.0.0 --rpc-bind-port=53000 --fee-address=KixW1oDMYMyabY6CdwS13fQuUwmUeXTPThF9s9JfDggbXSQGnZXkrP9LvcJtV9x7qb2pLsSobkXWXCrPsGGeC1V6VPBhPva --fee-amount=0.1
StandardInput=tty-force
TTYVHangup=yes
TTYPath=/dev/tty20
#TTYReset=yes
#RemainAfterExit=false
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

**Do not forget to change address to your wallet!**

9. Run service:

```
systemctl daemon-reload
systemctl enable dogemoned.service
systemctl start dogemoned.service
```

10. Check service status:

```
sudo systemctl status dogemoned.service
```
You should see seomething like this:
```
● dogemoned.service - dogemoned
   Loaded: loaded (/etc/systemd/system/dogemoned.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2020-10-17 19:24:47 CEST; 5s ago
     Docs: http://dogemone.online
 Main PID: 5368 (dogemoned)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/dogemoned.service
           └─5368 /home/dogemone/DOGEMONE/dogemoned --data-dir=/home/dogemone/DOGEMONE/.dogemone --log-file=/var/log/dogemoned --restricted-rpc --
Oct 17 19:24:47 dogemonenode.top systemd[1]: Started dogemoned.
lines 1-10/10 (END)
```

And check your Dogemone masternode from browser (don't forget to change server url to yours):
- http://dogemonenode.top:53000/feeaddress
- http://dogemonenode.top:53000/getinfo

You should see something like
```
{
  "fee_address": "KixW1oDMYMyabY6CdwS13fQuUwmUeXTPThF9s9JfDggbXSQGnZXkrP9LvcJtV9x7qb2pLsSobkXWXCrPsGGeC1V6VPBhPva",
  "fee_amount": 100000000000,
  "status": "OK"
}
```
and
```
{
  "already_generated_coins": "8741526.871278777480",
  "alt_blocks_count": 0,
  "block_major_version": 4,
  "contact": "",
  "cumulative_difficulty": 4061326490174723,
  "difficulty": 11531062982,
  "grey_peerlist_size": 3737,
  "height": 543420,
  "incoming_connections_count": 0,
  "last_known_block_index": 543419,
  "max_cumulative_block_size": 1423487,
  "min_fee": 100000000000,
  "next_reward": 4800694002995,
  "outgoing_connections_count": 5,
  "rpc_connections_count": 4,
  "start_time": 1602955812,
  "status": "OK",
  "top_block_hash": "50dff3d81e48b90168e0015009660587a17a396282d28011aad71c27f1488b65",
  "transactions_count": 619557,
  "transactions_pool_size": 5,
  "version": "1.7.6.957-6070815b",
  "white_peerlist_size": 268
}
```
respectively.

Also, you can connect to the node's output by `sudo conspy 20 #` if you install _conspy_.

## Update Dogemone node

1. Login to _dogemone_ user, go to 'DOGEMONE' folder:

```
su dogemone
cd DOGEMONE
```
2. Check current Dogemone version: `./dogemoned --version`
3. wget new Dogemone binaries from https://github.com/HashHound/Dogemone/releases/.
4. Stop Dogemone daemon: `sudo systemctl stop dogemoned.service`
5. Extract Dogemone binaries and replace old with a fresh one.
6. Start Dogemone daemon: `sudo systemctl start dogemoned.service` and check Dogemone daemon status `sudo systemctl status dogemoned.service` and version: `./dogemoned --version`
