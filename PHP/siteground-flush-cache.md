# SiteGround: Console Command to Flush Dynamic Cache

## Login with SSH
* Use web interface to create SSH key
* Copy private key into key.txt locally
* Load key.txt into bitvise client key manager
* Configure bitvise to use publickey+password

## Purge Cache over SSH
```sh
site-tools-client domain update id=1 flush_cache=1
```

Domain IDs can be listed using:
```sh
site-tools-client domain list -f id,name
```

## Purge Cache from another host
```sh
ssh example.com@gcam1098.siteground.biz -p 18765 "site-tools-client domain update id=1 flush_cache=1"
```

## Purge Cache with PHP

```php
<?php
header("Cache-Control: no-cache");
header("Content-Type: text/plain");
exec("site-tools-client domain update id=1 flush_cache=1");
echo("Purged " . time());
```

---

## Purge via cURL (old method)
This command executed on the surver will purge the dynamic cache:

```sh
curl -X PURGE http://127.0.0.1/ -H "Host: example.com"
```

### purge.php
```php
<?php
header("Cache-Control: no-cache");
if ($_POST['key'] == "PASSWORD123"){
	exec('curl -X PURGE http://127.0.0.1/ -H "Host: swharden.com"');
	echo "Purge: Successful";
} else {
	echo "ERROR: Invalid Key";
}
```

### Request
```
curl --data "key=PASSWORD123" https://swharden.com/purge/
```

### .htaccess
```
Header set Cache-Control "private"
```

### GitHub Action to Build and Deploy a Hugo site
```yaml
name: Website

on:
  workflow_dispatch:
  push: # only runs on commits to main branch (typically after a merge)
    branches:
      - main

jobs:
  deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
      - name: 🛒 Checkout
        uses: actions/checkout@v3
      - name: ✨ Setup Hugo
        env:
          HUGO_VERSION: 0.100.1
        run: |
          mkdir ~/hugo
          cd ~/hugo
          curl -L "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz" --output hugo.tar.gz
          tar -xvzf hugo.tar.gz
          sudo mv hugo /usr/local/bin
      - name: 🛠️ Build
        run: hugo
      - name: 🔐 Create Key File
        run: install -m 600 -D /dev/null ~/.ssh/id_rsa
      - name: 🔑 Populate Key
        run: echo "${{ secrets.PRIVATE_KEY }}" > ~/.ssh/id_rsa
      - name: 🚀 Upload
        run: rsync --archive --stats -e 'ssh -p 18765 -o StrictHostKeyChecking=no' public/ scottplot.net@ssh.scottplot.net:~/www/scottplot.net/public_html/ # no delete!
      - name: 🧹 Clear Cache
        run: ssh scottplot.net@ssh.scottplot.net -p 18765 "site-tools-client domain update id=1 flush_cache=1"
```
