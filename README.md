---
title: # "How to Serve a Load Balancer on a
  single server using Nginx and simple Python server (Without Using Docker)"
---

### Introduction

Let me introduce myself first, I am a graduate engineer with a degree in
Computer Science and I have experience with different tools and
technology for over 6 years now, that includes freelancing and
contracts. I am an highly motivated and enthusiastic person who seeks
for challenges and upskilling my knowledge all the time and that brings
me to my interest in cloud tech.

### About this Project

This project is all about practicing the basic concepts of the NGINX
server as a whole and implementing it on a live VPS over Digitalocean.

I have taken two small servers that are hosted on two different screens
as well as a Nginx server that is acting as a load balancer to both of
the web pages.

### Steps to Make it Happen

#### Acquiring Server and Setting Up Nginx

-   Firstly, purchase a VPS server from
    > [DigitalOcean](https://m.do.co/c/37aed4a4d685). You
    > will receive a credit of \$200 as you buy it for the first time.
    > Make sure you have purchased the Ubuntu server as it is best to
    > get you started on your linux journey.

    -   NOTE: You should create and associate SSH credentials to the
        > server. Store it in a safe place.

-   SSH into the server by navigating to the folder where the SSH key is
    > stored. Here's how you can do it,\
    > ssh -i \<SSH_File\> root@\<Server IP Address\>\
    > \
    > You need to replace \<SSH_File\> with the file name of the ssh
    > file and \<Server IP Address\> with the server IP that can be
    > found under Digital Ocean server dashboard\
    > ![](./image4.png){width="6.5in"
    > height="2.8333333333333335in"}\
    > *Arrow is indicating server IP Address on a DigitalOcean Droplet
    > area.\
    > *\
    > This is how it looks typically, *\
    > *![](./image1.png){width="8.166666666666666in"
    > height="0.2604166666666667in"}\
    > \
    > Next, you will need to enter the passphrase that you've given
    > while creating the SSH file and login to the server.

-   Right after you login, install nginx server by following the steps
    > mentioned here -
    > [[https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04]{.underline}](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04)\
    > Or,\
    > [[https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/]{.underline}](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)

-   Once installed, verify it by running the following command,\
    > *systemctl status nginx*\
    > \
    > It should give the following output\
    > \
    > ![](./image3.png){width="9.239583333333334in"
    > height="2.5729166666666665in"}

    -   If you are facing issues like unable to start nginx or something
        > similar, you should try killing the service and starting it
        > again. For that, do the following,

        -   Firstly, check for configuration error on the nginx.conf
            > file by typing\
            > *sudo nginx -t*

        -   If everything is fine, kill the service for nginx, (or any
            > underlying zombie nginx service)\
            > *sudo pkill nginx*

        -   Now start nginx server again by typing,\
            > *nginx or systemctl start nginx*

-   If everything is looking good curl into the server ip to see the
    > default page of nginx. From the browser, you will see something
    > like this,\
    > \
    > ![](./image2.png){width="6.177083333333333in"
    > height="1.6458333333333333in"}

-   YAY! You've set up Nginx.

#### Setting up the web pages

-   Install git on your server now by running *sudo apt install git*

-   Now navigate to */var/www*

-   Next, run *git clone
    > [[https://github.com/poulav/nginx-website.git]{.underline}](https://github.com/poulav/nginx-website.git)*
    > to clone my git repository containing the necessary files.

    -   If you do not want to use my files, simply create two HTML files
        > both with the name as index.html in two different folders.

    -   You need to navigate to each of the folders and run *sudo
        > python3 -m http.server 3001* , here 3001 is the port number.
        > You should use two different ports for the two different html
        > pages to run them on the server.

-   On my folder context, once I will start the server on
    > *nginx_website,* and another one under
    > *nginx_website/HostingDirectly/fruits*

-   To keep the servers running in parallel we will need to make use of
    > screens on linux, for that we need to do the following,

    -   [First, navigate into the *nginx_website/HostingDirectly* folder
        > and run *screen -S \<screen_name\>* to start a command line
        > screen(make sure you replace the \<screen_name\> with an
        > appropriate screen name) there then start the server using
        > *python3 -m http.server 3001*]{.mark}

    -   [Next, navigate inside fruits and do the same with a different
        > screen name and server port. I will be using screen2 and 3002
        > as the screen name and port number respectively.\
        > \
        > NOTE: To exit from one screen you will need to press *ctrl + a
        > and then d* from your keyboard to detach from screen. And, to
        > check the process: *screen -r \<screen_name\>.*]{.mark}

-   [You have covered the second step.]{.mark}

#### Setting up the NGINX Server

-   Navigate to /etc/nginx/ and then open nginx.conf by typing *sudo vim
    > nginx.conf.* (I will be using vim as my editor, you can use
    > anything.)

-   Here's a sample nginx.conf complete file,

> *user nginx;*
>
> *worker_processes auto;*
>
> *error_log /var/log/nginx/error.log notice;*
>
> *pid /var/run/nginx.pid;*
>
> *events {*
>
> *worker_connections 1024;*
>
> *}*
>
> *http {*
>
> *include /etc/nginx/mime.types;*
>
> *default_type application/octet-stream;*
>
> *\### Here backendserver is a variable/name of the group of upstream
> servers*
>
> *upstream backendserver {*
>
> *server 127.0.0.1:3001;*
>
> *server 127.0.0.1:3002;*
>
> *}*
>
> *\### Load BALANCER / Reverse Proxy*
>
> *server {*
>
> *server_name ng.poulav.dev;*
>
> *location / {*
>
> *proxy_pass
> [[http://backendserver]{.underline}](http://backendserver); \# Here
> the variable sits in that maps the servers mentioned inside the
> upstream variable.*
>
> *}*
>
> *listen 443 ssl; \# managed by Certbot*
>
> *ssl_certificate /etc/letsencrypt/live/ng.poulav.dev/fullchain.pem; \#
> managed by Certbot*
>
> *ssl_certificate_key /etc/letsencrypt/live/ng.poulav.dev/privkey.pem;
> \# managed by Certbot*
>
> *include /etc/letsencrypt/options-ssl-nginx.conf; \# managed by
> Certbot*
>
> *ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; \# managed by Certbot*
>
> *}*
>
> *\### SERVER 1*
>
> *server {*
>
> *server_name s1.ng.poulav.dev;*
>
> *location / {*
>
> *proxy_pass https://\<SERVER_IP\>:3001; #The \<SERVER_IP\> has to be
> replaced with your original IP of the server that you can find in the
> dashboard of DigitalOcean.*
>
> *proxy_http_version 1.1;*
>
> *proxy_set_header Upgrade \$http_upgrade;*
>
> *proxy_set_header Host \$host;*
>
> *proxy_cache_bypass \$http_upgrade;*
>
> *}*
>
> *listen 443 ssl; \# managed by Certbot*
>
> *ssl_certificate /etc/letsencrypt/live/s1.ng.poulav.dev/fullchain.pem;
> \# managed by Certbot*
>
> *ssl_certificate_key
> /etc/letsencrypt/live/s1.ng.poulav.dev/privkey.pem; \# managed by
> Certbot*
>
> *include /etc/letsencrypt/options-ssl-nginx.conf; \# managed by
> Certbot*
>
> *ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; \# managed by Certbot*
>
> *}*
>
> *\### SERVER 2*
>
> *server {*
>
> *server_name s2.ng.poulav.dev;*
>
> *location / {*
>
> *proxy_pass https://\<SERVER_IP\>:3002; #The \<SERVER_IP\> has to be
> replaced with your original IP of the server that you can find in the
> dashboard of DigitalOcean.*
>
> *proxy_http_version 1.1;*
>
> *proxy_set_header Upgrade \$http_upgrade;*
>
> *proxy_set_header Connection \'upgrade\';*
>
> *proxy_set_header Host \$host;*
>
> *proxy_cache_bypass \$http_upgrade;*
>
> *}*
>
> *listen 443 ssl; \# managed by Certbot*
>
> *ssl_certificate /etc/letsencrypt/live/s1.ng.poulav.dev/fullchain.pem;
> \# managed by Certbot*
>
> *ssl_certificate_key
> /etc/letsencrypt/live/s1.ng.poulav.dev/privkey.pem; \# managed by
> Certbot*
>
> *include /etc/letsencrypt/options-ssl-nginx.conf; \# managed by
> Certbot*
>
> *ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; \# managed by Certbot*
>
> *}*
>
> *log_format main \'\$remote_addr - \$remote_user \[\$time_local\]
> \"\$request\" \'*
>
> *\'\$status \$body_bytes_sent \"\$http_referer\" \'*
>
> *\'\"\$http_user_agent\" \"\$http_x_forwarded_for\"\';*
>
> *access_log /var/log/nginx/access.log main;*
>
> *sendfile on;*
>
> *#tcp_nopush on;*
>
> *keepalive_timeout 65;*
>
> *#gzip on;*
>
> *include /etc/nginx/conf.d/\*.conf;*
>
> *server {*
>
> *if (\$host = ng.poulav.dev) {*
>
> *return 301 https://\$host\$request_uri;*
>
> *} \# managed by Certbot*
>
> *listen 80;*
>
> *server_name ng.poulav.dev;*
>
> *return 404; \# managed by Certbot*
>
> *}*
>
> *server {*
>
> *if (\$host = s1.ng.poulav.dev) {*
>
> *return 301 https://\$host\$request_uri;*
>
> *} \# managed by Certbot*
>
> *server_name s1.ng.poulav.dev;*
>
> *listen 80;*
>
> *return 404; \# managed by Certbot*
>
> *}*
>
> *server {*
>
> *if (\$host = s2.ng.poulav.dev) {*
>
> *return 301 https://\$host\$request_uri;*
>
> *} \# managed by Certbot*
>
> *server_name s2.ng.poulav.dev;*
>
> *listen 80;*
>
> *return 404; \# managed by Certbot*
>
> *}}*

You must have seen a lot of new things in this configuration file above.
To understand it better, you will need to understand the basics of Nginx
Server and its configuration which you can get it from here -
[[https://youtu.be/7VAI73roXaY]{.underline}](https://youtu.be/7VAI73roXaY)

Here's an article that explains how to set up Reverse proxy on a
localhost setup,
[[https://phoenixnap.com/kb/docker-nginx-reverse-proxy]{.underline}](https://phoenixnap.com/kb/docker-nginx-reverse-proxy)

In short, the http section contains all the proxy and server
configurations along with SSL information.

The backendserver upstream variable itself plays a major role.

Now the servers are configured.

If you closely look at the configuration file, you will see a lot of
lines with a comment on them as *managed by Certbot.*

That is due to it being generated by LetsEncrypt's certificate bot that
is used to associate SSL with the servers. For this, you will simply
need to follow the following steps mentioned here -
[[https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal&tab=standard]{.underline}](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal&tab=standard)

That's it! All DONE...
