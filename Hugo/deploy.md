# How to Deploy a Hugo Site

## rsync

The advantage of `rsync` over other methods is that it's smart enough to only upload _modified_ files. This doesn't matter for small sites, but it's super helpful for large ones.

This command deploys a folder over SSH using `rsync` on Ubuntu through WSL.

```bash
sudo rsync \
  --archive \
  --progress \
  --delete \
  --stats \
  -e 'ssh -p 18765 -i /mnt/c/Users/scott/Documents/important/certs/2022-03-12-id_rsa_no-passphrase.private' \
  '/mnt/c/Users/scott/Documents/GitHub/Csharp-Data-Visualization/website/output/' \
  u186-exgapiqqc7gg@swharden.com:/home/customer/www/swharden.com/public_html/csdv;
```