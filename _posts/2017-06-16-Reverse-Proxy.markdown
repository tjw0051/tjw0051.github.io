---
layout:     post
title:      "Hosting Multiple Websites on the same Server"
subtitle:   "Reverse Proxies with Nginx"
date:       2017-08-30 17:08:00
author:     "Tom"
header-img: "img/2017-08-30_cover.jpg"
header-mask: 0.5
catalog: true
tags:
    - Go
    - shell
    - Project Management
---

# Overview

For setting up small projects you often find that a single server is sufficiently powerful to serve websites or services that are infrequently used. For setting up a single website on a server, you might have spun up a node or apache server, set it to listen on port 80, and forwarded your domain name (e.g. mywebsite.com) to your server's IP address (27.0.3.4). When someone visits your website, they browse to mywebsite.com, send a GET or other request to your web server, which returns web pages for your website. 

It becomes apparent that this is a problem if you have multiple domains pointed to the same server. For example if you have 2 node.js servers running, the first serving pages for mywebsite.com, and the second for notmywebsite.com, the server will not know which website the user typed into their browser, as both domain names point to your server. Furthermore you can't bind both servers to port 80 in most cases. The solution is a reverse-proxy.

Nginx is a hugely popular http server, whos first introduction to most people is as a proxy server for our situation. A proxy server is a server that [acts as an intermediary for requests from clients seeking resources from other servers.](https://en.wikipedia.org/wiki/Proxy_server). In this case our requests from clients are our websites (mywebsite.com and notmywebsite.com) and the resources are the web pages our two apache/node/etc instances are serving.

---

# Installing Nginx

This installation will assume you are installing Nginx on Ubuntu 14.04 or 16.04. If that is not the case it is recommended that you find a seperate installation guide for this step to avoid any problems.

Installing Nginx on a fresh ubuntu 14.04 or 16.04 should be straightforward. First we update the package manager then install nginx:
```sh
sudo apt-get update
sudo apt-get install nginx-full
```

# Configuring Nginx

For this example we'll be configuring two websites, but it should be trivial to configure more.

Nginx stores host configurations in `/etc/nginx/sites-available/`. After installation, a default configuration will be created in this directory. Delete this file as we'll be creating our own configurations:

```sh
sudo rm /etc/nginx/sites-enabled/default
```

Next create two files in this directory for our two websites (substitute the names for those of your websites):

```sh
sudo touch /etc/nginx/sites-available/mywebsite.com
sudo touch /etc/nginx/sites-available/notmywebsite.com
```

Before the next step you'll need to configure your server's ports. if you've set up a web server you might be exposing it on port 80. For both applications, change these to ports that are NOT 80 and not already in use. Picking ports > 2000 is a safe bet. The servers also cannot use the same port. For my node.js servers, i'll be setting the following ports:

```
mywebsite.com    -> :8081
notmywebsite.com -> :8082
```

In node (using express) this would be done by `app.listen(8081)` for my first server, and `app.listen(8082)` for the second server. The reason for this will become clear in the next step.

# Defining Nginx Hosts

Next we're going to edit the host files we created earlier and add our configuration. Open `/etc/nginx/sites-available/mywebsite.com` using your favourite text editor:
```
sudo nano /etc/nginx/sites-available/mywebsite.com
```

Input the following configuration:
```sh
server {
    listen 80;

    server_name mywebsite.com www.mywebsite.com;

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Change the `server_name` line to your website address, with 2 versions, one with and one without the `www`. Change the `proxy_pass` line where it says `8081` to whichever port you decided to use for your first website. Save the file and close it. 

Next open up our second host file:
```
sudo nano /etc/nginx/sites-available/notmywebsite.com
```

...and input the following:

```sh
server {
    listen 80;

    server_name notmywebsite.com www.notmywebsite.com;

    location / {
        proxy_pass http://127.0.0.1:8082;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Again change the `server_name` line to match those for your second website, and change the port of the `proxy_pass` line from `8082` to whatever the port is that you set for your second server. Save and close the file.

# Loading the New Configuration

Nginx has a handy tool for testing if a configuration has been created correctly. Run the following command:
```sh
sudo nginx -t
```

If the output says the syntax is OK and test is successful then we can proceed. If not, go back through the previous step to see if theres anything you might have missed.

Next we reload the Nginx service to use our new configuration:

```sh
sudo systemctl reload nginx
```

Our server should now be set up correctly. If your web servers aren't already running, run them now.

# Configuring DNS

Next you'll need to go to your DNS and create A records for your domain names that point to your server's address. This is going to be specific to whatever DNS you're using but here's an example of my setup from DigitalOcean to give you an idea:

![MyWebsite.com](http://imgur.com/1wOOU9Q.png)

![NotMyWebsite.com](http://imgur.com/hZKveYK.png)

In the examples above, the records for both `mywebsite.com` and `notmywebsite.com` point to my server's IP address `178.60.100.102`.

# Test It

If you've set up nginx and reloaded the configuration, started your web servers, and forwarded your domain names to point to your server's address, its time to test it out. Open a web browser and navigate to your two websites. You should be served the correct web pages for the website you accessed. If not, re-check the configuration in the above steps and make sure you've set the correct ports. 

It may also take some time for your DNS records to be updated, so wait 5 minutes then try again. Additionally make sure that if you have any caching layers (e.g. Cloudflare Always Online™ or other), purge the cache so that the changes are propogated. 

---

# Conclusion

I hope this guide has helped you set up nginx and save a little money by hosting multiple websites on the same server. 

If you're interested in going a step further and securing your server with HTTPS/TLS I highly recommend [this](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04) guide by DigitalOcean.


—— Tom 2017.08
