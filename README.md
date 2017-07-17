# deploy-react-app

## git deploy
SSH into droplet

### create a bare repo
```
cd ~ 
mkdir {site}.git && cd {site}.git
git init --bare
```

### add post receive
```
cat > hooks/post-receive
```

Paste the following while substituting {VARIABLES} with your own:
```
#!/bin/sh
export GIT_WORK_TREE="{SITE_ABSOLUTE_PATH}"
export NODE_VERSION="6.11"

git --work-tree="$GIT_WORK_TREE" --git-dir={REPO_ABSOLUTE_PATH} checkout -f

. $HOME/.nvm/nvm.sh
nvm install $NODE_VERSION
nvm use $NODE_VERSION
nvm current

cd "$GIT_WORK_TREE" 
npm install
npm run build
```

Then make the post receive executable
```
chmod +x post-receive
```

### local machine
```
cd {REPO}
git remote add live ssh://root@shannonarcher.me/{REPO_ABSOLUTE_PATH}
git push live master
```

## add records to digital ocean
Add an `A` record and direct to the droplet.
```
HOSTNAME: {SUBDOMAIN}.{DOMAIN}.{TLD}
WILL DIRECT TO: {DROPLET}
```

Create a `CNAME` record and alias for the `A` record you've just created.
```
HOSTNAME: *.{SUBDOMAIN}
IS AN ALIAS OF: {SUBDOMAIN}.{DOMAIN}.{TLD}
```

## create new site in nginx
```
nano /etc/nginx/conf.d/{SUBDOMAIN}.{DOMAIN}.{TLD}.conf
```

Paste the following while substituting {VARIABLES} with your own:
```
server {
    listen 80;
    server_name {SUBDOMAIN}.{DOMAIN}.{TLD};
    location / {
        proxy_pass http://localhost:{PORT};
    }
}
```

## create node server (app.js) to serve index
```
const express = require('express');
const path = require('path');
const app = express();

app.use(express.static(path.join(__dirname, 'build')));

app.get('/', function (req, res) {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});

app.listen({PORT});
```

## set service to run
```
pm2 start /root/{REPO}/app.js
pm2 save
```