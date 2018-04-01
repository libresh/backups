/!\ deprecated in favor of https://github.com/libresh/borg-server

If you use it and want to maintain it, let me know, I'll transfer ownership to you or give you rights to manage it here.

# backups

It backups domains \o/.
This is a full backup system for LibrePaaS.

## How it works

It goes to every domain, run the `pre-backup` script if present (it is usually a database dump). Then it uses duplicity to send it remotely (to an ssh endpoint for instance).

## How to use

First make sure you have [libre.sh](https://github.com/indiehosters/libre.sh) installed.

And then:

```bash
cd /system
git clone https://github.com/indiehosters/backups
cd backups && ./scripts/install
# edit the env with the BACKUP_DESTINATION=user@host:port
# add the ssh key for backup container (/system/backups/conf/root/.ssh/id_rsa.pub) to autorized_hosts on the backup server
# add the remote key to known hosts:
docker run -it -v /system/backups/conf/root:/root debian ssh -o "StrictHostKeyChecking no" -o "BatchMode yes" -o "HostKeyAlgorithms=ssh-rsa" $USER@$HOST -p$PORT exit
systemctl start s@backups
systemctl enable s@backups
```

## How to recover

Imagine your server just burn, you have to follow the following procedure:

 - find the backed up PASSPHRASE of the lost server
 - fire a new server
 - configure it with the PASSPHRASE
 - configure the remote server to accept your ssh key

```bash
source /etc/environment
# List all backups
/bin/docker run \
  -it \
  --rm \
  --name backup-$domain \
  -e PASSPHRASE \
  -h backup.container \
  -v /root:/root \
  indiehosters/backups \
    collection-status \
    sftp://root@IP//shared/$domain

#recover 7 days ago
/bin/docker run \
  -it \
  --rm \
  --name backup-$domain \
  -e PASSPHRASE \
  -h backup.container \
  -v /root:/root \
  -v /data/domains/$domain:/backup \
  indiehosters/backups \
    -t 7D \
    sftp://root@IP//shared/$domain \
    /backup
```

## Advanced setup

### Configure an SSH container on the BACKUP_DESTINATION to receive the backups

### Use public/private gpg keys to encrypt the backups instead of a passphrase.
