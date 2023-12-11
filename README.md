# Docker-Stack-Ghost-MySQL
 A docker compose stack with the blogging platform Ghost CMS and a MySQL Database. The docker stack will be deployed in `production` mode.
 <br> 

## What is Ghost CMS?
Ghost CMS is a blogging platform where most you as a writer can focus more on writing than coding or configuring the settings. I personally use Ghost for my own website/blogging needs. To learn more visit the offical [Ghost Page](https://ghost.org/). 

## Requirements
Using this repo will require a few things:
1. Docker Destop installed, [how to here](https://docs.docker.com/desktop/).
2. Nginx Proxy Manager Installed, [how to here](https://github.com/ravencore17/Docker-Stack-Nginx-MariaDB).
3. A domain or subdomain.
4. STMP settings for your transactional emails: signups, password resets and admin invites.
5. Optional - Mailgun settings for your bulk emails, Newsletters.

## Install

### 1. Get the Docker Compose
Clone the repo, download the docker-compose.yml or copy the following into a YAML file.
```yaml
version: "3.3"

networks:
  frontend:
    external: true
  backend:
  
services:
  ghost:
    image: ghost:latest
    restart: always
    container_name: ghost-app
    depends_on:
      - db
    environment:
      url: https://domain.com
      database__client: mysql
      database__connection__host: ghost-db
      database__connection__port: 3306
      database__connection__user: ghostuser
      database__connection__password: DBPASSWORD
      database__connection__database: ghost
      mail__transport: SMTP
      mail__options__host: host.domain.com
      mail__options__port: 587
      mail__options__auth__user: smtp.username
      mail__options__auth__pass: smtp.password
      mail__from: Example Corpo <mail@domain.com>
    volumes:
      - ./content:/var/lib/ghost/content
    networks:
      - frontend
      - backend

  db:
    image: mysql:latest
    restart: always
    container_name: ghost-db
    environment:
      MYSQL_ROOT_PASSWORD: DBPASSWORDROOT
      MYSQL_DATABASE: ghost
      MYSQL_USER: ghostuser
      MYSQL_PASSWORD: DBPASSWORD
    volumes:
      - ./db:/var/lib/mysql
    networks:
      - backend
```
Note: if you are downloading the docker-compose.yml or making your own, its in your own best interest to nest that YAML file in its own folder for simplier `docker compose up -d`.<br>
Below we are going further into variables and general configurations.

### 2. Networks
```yaml
networks:
  frontend:
    external: true
  backend:
```
We have two networks defined here: `frontend` and `backend`. You'll notice that `frontend ` is external, that's so that it can talk to the NGINX Proxy Manager reverse proxy that was set up eariler. <br>
The network `backend` is strickly for the Ghost application to talk to its MySQL database. The `external: true` hash states the network already exist and for the docker service to assign the container to it that network.
<br>

### 3. Ghost CMS Application
```yaml
services:
  ghost:
    image: ghost:latest
    restart: always
    container_name: ghost-app
    depends_on:
      - db
    environment:
      url: https://domain.com
      database__client: mysql
      database__connection__host: ghost-db
      database__connection__port: 3306
      database__connection__user: ghostuser
      database__connection__password: DBPASSWORD
      database__connection__database: ghost
      mail__transport: SMTP
      mail__options__host: host.domain.com
      mail__options__port: 587
      mail__options__auth__user: smtp.username
      mail__options__auth__pass: smtp.password
      mail__from: Example Corpo <mail@domain.com>
    volumes:
      - ./content:/var/lib/ghost/content
    networks:
      - frontend
      - backend
```
Lets break it down even more:<br><br>
```yaml
  depends_on:
    - db
```
The docker service will start the database container before the main app. This is to ensure that the dependenceis needed for the app are avail when the app is spun up.<br><br>
```yaml
   environment:
      url: https://domain.com
```
You'll want to put your sub domain or domain here, whatever the exact address is of your blog. Make sure you state your connection type: HTTP or HTTPS, just for the record you should always go for HTTPS, as it's more secure.<br><br>

```yaml
      database__client: mysql
      database__connection__host: ghost-db
      database__connection__port: 3306
      database__connection__user: ghostuser
      database__connection__password: DBPASSWORD
      database__connection__database: ghost
```
This section is the client configuration so that `ghost-app` can access your database container, `ghost-db` securely.<br>
`database_connection_host: ghost-db` Where our ghost database is hosted, in this case the MySQL container `ghost-db`. <br>
`database__connection__port: 3306` Is the default MySQL connection port.<br>
`database__connection__user: ghostuser` Defines the user of the database<br>
`database__connection__password: DBPASSWORD` The password associated with the user <br>
**Make sure you change the `database_connection_password` environment variable to something more secure.**<br>
`database__connection__database: ghost` The name of the database inside MySQL.<br><br>

```yaml
      mail__transport: SMTP
      mail__options__host: host.domain.com
      mail__options__port: 587
      mail__options__auth__user: smtp.username
      mail__options__auth__pass: smtp.password
      mail__from: Example Corpo <mail@domain.com>
```
This section is going to be dealing with the transactional emails. Password resets, newsletter signups and collaberation invites are all examples of transactional emails. Ghost does bulk email services through the Mailgun API, which can also act as a transactional service.  However this configuration based on my own hosting does not use the Mailgun API. You can add the Mailgun feature later on through the Ghost admin panel.<br><br>

`mail__transport: SMTP` Transactional emails are always SMTP.<br>
`mail__options__host: host.domain.com` Your email service STMP host domain.<br>
`mail__options__port: 587` SMTP emails are usually sent on the 587 port.<br>
`mail__options__auth__user: smtp.username` The username for your SMTP service.<br>
`mail__options__auth__pass: smtp.password` Password for your SMTP username<br>
`mail__from: Example Corpo <mail@domain.com>` What the person receiving would see it's from.<br>
**An example configuration would look like this if you were using iCloud**
```yaml
      mail__transport: SMTP
      mail__options__host: smtp.mail.me.com
      mail__options__port: 587
      mail__options__auth__user: noreply.you@icloud.com
      mail__options__auth__pass: cwla-ktrm-pzou-dznn #App specific password
      mail__from: your.domain.com <noreply.you@icloud.com>
```
