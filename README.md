# What is XWiki

[XWiki](http://xwiki.org) is a free wiki software platform written in Java with a design emphasis on extensibility. XWiki is an enterprise wiki. It includes WYSIWYG editing, OpenDocument based document import/export, semantic annotations and tagging, and advanced permissions management.

As an application wiki, XWiki allows for the storing of structured data and the execution of server side script within the wiki interface. Scripting languages including Velocity, Groovy, Python, Ruby and PHP can be written directly into wiki pages using wiki macros. User-created data structures can be defined in wiki documents and instances of those structures can be attached to wiki documents, stored in a database, and queried using either Hibernate query language or XWiki's own query language.

[XWiki.org's extension wiki](http://extensions.xwiki.org) is home to XWiki extensions ranging from [code snippets](http://snippets.xwiki.org) which can be pasted into wiki pages to loadable core modules. Many of XWiki Enterprise's features are provided by extensions which are bundled with it.

![logo](http://www.xwiki.org/xwiki/bin/view/Main/Logo?xpage=plain&act=svg&finput=logo-xwikiorange.svg&foutput=logo-xwikiorange.png&width=200)

# Introduction

The goal is to provide a production-ready XWiki system running in Docker. This is why:
- The OS is based on Debian and not on some smaller-footprint distribution like Alpine
- Several containers are used with Docker Compose: one for the DB and another for XWiki + Servlet container. This allows the ability to run them on different machines for example. 

# How to use this image

[![](https://images.microbadger.com/badges/version/xwiki/xwiki-mysql-tomcat.svg)](https://microbadger.com/images/xwiki/xwiki-mysql-tomcat) [![](https://images.microbadger.com/badges/image/xwiki/xwiki-mysql-tomcat.svg)](https://microbadger.com/images/xwiki/xwiki-mysql-tomcat)

You should first install [Docker](https://www.docker.com/) on your machine.

Then there are several options:

1.  Pull the xwiki image from DockerHub.
2.  Get the [sources of this project](https://github.com/xwiki-contrib/docker-xwiki) and build them.

## Pulling existing image

You need to run 2 containers:
- One for the XWiki image
- One for the database image to which XWiki connects to

The simplest is to use the Docker Compose file we provide. Run the following steps:
- `wget https://raw.githubusercontent.com/xwiki-contrib/docker-xwiki/master/xwiki-mysql-tomcat/docker-compose-using.yml`
  - If you're not using the `latest` tag then use the corresponding GitHub branch/tag. For example for the `8.x` branch: `wget https://raw.githubusercontent.com/xwiki-contrib/docker-xwiki/8.x/xwiki-mysql-tomcat/docker-compose-using.yml`
- You can edit the compose file retrieved to change the default username/password and other environment variables.
- `docker-compose up`

For reference here's a minimal Docker Compose file using MySQL that you could use as an example (full example
[here](https://github.com/xwiki-contrib/docker-xwiki/blob/master/xwiki-mysql-tomcat/docker-compose-using.yml)):

```yaml
version: '2'
services:
  web:
    image: "xwiki/xwiki-mysql-tomcat:latest"
    depends_on:
      - db
    ports:
      - "8080:8080"
    volumes:
      - xwiki-data:/var/lib/xwiki
    environment:
      - MYSQL_USER=xwiki
      - MYSQL_PASSWORD=xwiki
  db:
    image: "mysql:5"
    volumes:
      - ./mysql/xwiki.cnf:/etc/mysql/conf.d/xwiki.cnf
      - mysql-data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=xwiki
      - MYSQL_USER=xwiki
      - MYSQL_PASSWORD=xwiki
      - MYSQL_DATABASE=xwiki
volumes:
  mysql-data: {}
  xwiki-data: {}
```

## Building

This allows you to rebuild the XWiki docker image locally. Here are the steps:

-	Install Git and run `git clone https://github.com/xwiki-contrib/docker-xwiki.git` or download the sources from the GitHub UI. Then choose the branch or tag that you wish to use:
	-	The `master`branch will get you the latest released version of XWiki
	-	The `8.x` branch will get you the latest released version of XWiki for the 8.x cycle
	-	The `8.4.4` tag will get you exactly XWiki 8.4.4.
	-	etc.
-	Go the directory corresponding to the configuration you wish to build, for example: `cd xwiki-mysql-tomcat`.
-	Run `docker-compose up`
-	Start a browser and point it to `http://localhost:8080`

Note that if you want to set a custom version of XWiki you can checkout `master` and edit the `env` file and set the values you need in there. It's also possible to override them on the command line with `docker-compose run -e "XWIKI_VERSION=8.4.4"`.

Note that `docker-compose up` will automatically build the XWiki image on the first run. If you need to rebuild it you can issue `docker-compose up --build`. You can also build the image with `docker build . -t xwiki-mysql-tomcat:latest` for example.

# Details for xwiki-mysql-tomcat

## Configuration Options

The first time you create a container out of the xwiki image, a shell script (`/usr/local/bin/start_xwiki.sh`) is executed in the container to setup some configuration. The following environment variables can be passed:

-	`MYSQL_USER`: The MySQL user name used by XWiki to read/write to the DB.
-	`MYSQL_PASSWORD`: The MySQL user password used by XWiki to read/write to the DB.

## Miscellaneous

Volumes:

-	Two volumes are created:
	-	A volume named `<prefix>_mysql-data` that contains the database data.
	-	A volume named `<prefix>_xwiki-data` that contains XWiki's permanent directory.
-	To find out where those volumes are located on your local host machine you can inspect them with `docker volume inspect <volume name>`. To find the volume name, you can list all volumes with `docker volume ls`.

-	Note that on Mac OSX, Docker runs inside the xhyve VM and thus the paths you get when inspecting the volumes are relative to this. Thus, you need to get into that VM if you need to access the volume data.

MySQL:

-	To issue some mysql commands:
	-	Find the container id with `docker ps`
	-	Execute bash in the mysql container: `docker exec -it <containerid> bash -l`
	-	Once inside the mysql container execute the `mysql` command: `mysql --user=xwiki --password=xwiki`

# License

XWiki is licensed under the [LGPL 2.1](https://github.com/xwiki-contrib/docker-xwiki/blob/master/LICENSE).

The Dockerfile repository is also licensed under the [LGPL 2.1](https://github.com/xwiki-contrib/docker-xwiki/blob/master/LICENSE).

# Support

* If you wish to raise an issue or an idea of improvement use [XWiki Docker JIRA project](http://jira.xwiki.org/browse/XDOCKER)
* If you have questions, use the [XWiki Users Mailing List/Forum](http://dev.xwiki.org/xwiki/bin/view/Community/MailingLists) or use the [XWiki IRC channel](http://dev.xwiki.org/xwiki/bin/view/Community/IRC)

# Contribute

* If you wish to help out on the code, please send Pull Requests on [XWiki Docker GitHub project](https://github.com/xwiki-contrib/docker-xwiki)

# Credits

* Created by Vincent Massol
* Contributions from Ludovic Dubost, Jean Simard
* Some code was copied from https://github.com/ThomasSteinbach/docker_xwiki. Thank you Thomas Steinbach
