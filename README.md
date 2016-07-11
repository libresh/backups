# backups

It backups domains \o/.
This is a full backup system for LibrePaaS.

## How it works

It goes to every domain, run the `pre-backup` script if present (it is usually a database dump). Then it uses duplicity to save it locally and encrypted with the gpg key created during the install procedure.

At the end, it exposes the `local and encrypted backup folder` in read only to ssh. Another server can come and pull backups to increase redundancy.

## How to use

First make sure you have [LibrePaaS](https://github.com/indiehosters/LibrePaaS) installed.

And then:

```bash
cd /system
git clone https://github.com/indiehosters/backups
cd backups && ./scripts/install
#follow instructions
systemctl start s@backups
systemctl enable s@backups
```

## How to recover

Imagine your server just burn, you have to follow the following procedure:

 - find the backed up gpg key of the lost server
 - make sure the redundant server exposes the backups with ssh
 - fire a new server
 - configure it with the gpg key
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
