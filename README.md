# Deployment

just trying to consolidate all the stuff i end up googling everytime i deploy something on ubuntu.
credit to DO and all other articles i reference from. 

### links
- [Nodejs Deployment][l1] 
- [Git Branch Deployment][l2]

### ubuntu installing necessary packages
```sh
sudo apt-get update
sudo apt-get install git
cd ~ wget https://nodejs.org/dist/v4.2.3/node-v4.2.3-linux-x64.tar.gz
mkdir node
tar xvf node-v*.tar.?z --strip-components=1 -C ./node
cd ~ && rm -rf node-v*
mkdir node/etc
echo 'prefix=/usr/local' > node/etc/npmrc
sudo mv node /opt/
sudo chown -R root: /opt/node
sudo ln -s /opt/node/bin/node /usr/local/bin/node
sudo ln -s /opt/node/bin/npm /usr/local/bin/npm
```

### Git Branch Deployment commands Checklist
#### Server
```sh
cd /var/source-code.git
git init --bare
cd hooks && touch post-receive
echo
"#!/bin/sh
git --work-tree=/var/www/livefolder --git-dir=/var/source-code.git checkout -f" > post-receive
chmod +x post-receive
```
#### local
```sh
git remote add live ssh://user@server:/var/source-code.git
git add . && commit -m "pushing live"
git push live master 
```

### Correct Rights to a live folder
```sh
sudo chown -R $USER:$USER /var/www/website-folder
sudo chmod -R 755 /var/www
```

### Nginx Set up
```sh
sudo apt-get install nginx
nano /etc/nginx/sites-available/default
```
add this :
```js
server {
    listen 80;
    server_name livedomain.com;
    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
```sh
sudo service nginx restart
```
This assumes your express.js somewhere says : 
```js
app.listen("localhost",3001);
```

### Pm2 set up
```sh
sudo npm install pm2 -g
```
Start Process & Restart it on file change (Deployment) [more][l3]
```sh
pm2 start app.js
```
Sample Watch File:  _process.json_
```json
{
  "apps" : [{
    "script"      : "index.js",
    "watch"       : true,
    "ignore_watch" : ["logs","node_modules","values"],
    "env": {
      "NODE_ENV": "production",
    }
  }]
}
```
Setup procedure :
```sh
pm2 start process.json
```

Set it up for Starting on server restart
```sh
pm2 startup
```
Save the current proccesses to see them start
```sh
pm2 save
```
useful commands :
```sh
pm2 list
pm2 show [id]
pm2 logs
pm2 monit
```

Further readings:
* [Monitoring][l4]
* Papertrail
* Winston
* [SSL HTTPS][l5]
* [Free SSL Let's Encrypt][l8]
* [Migrating from Apache -> Nginx][l6]
* [Server Blocks in Ngninx][l7]
* [Node_ENV variables][l9]


[l1]: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-14-04
[l2]:https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps
[l3]:http://pm2.keymetrics.io/docs/usage/watch-and-restart/
[l4]:http://pm2.keymetrics.io/docs/usage/monitoring/
[l6]:https://www.digitalocean.com/community/tutorials/how-to-migrate-from-an-apache-web-server-to-nginx-on-an-ubuntu-vps
[l5]:https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority
[l7]:https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-14-04-lts
[l8]:https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04
[l9]:http://www.hacksparrow.com/running-express-js-in-production-mode.html
