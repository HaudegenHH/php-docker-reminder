## The system that we are going to create consists of:
- Nginx
- PHP
- MySQL 
- Redis

..or to use the correct docker terminology: the "network" that we are going to create.

If you like a mental reference to compare this against, then think of sth like a preconfigured stack, such as the LAMP stack. 
This is composed of an Apache server, MySQL database, and PHP running on a Linux OS.
You can swap the OS from Linux to Mac and it becomes a MAMP stack, or for Windows: WAMP
..but with Docker however this network that we create will be portable and able to be spun up on any operating system.
So you can forget about that consideration and concentrate on the component parts, these will be all individual containers!

We swapped Apache for Nginx. And i have added Redis, which is mainly used for caching.

---

1. Starting with the creation of the docker-compose.yaml

With docker you have think of your collective application as a network of services, and each of those services is its own individual containerized component. In order to network them and make them communicate with each other we need to have some kind of mechanism. Luckily for us developers we can do this with code, more specifically with a file called a docker compose file. 

- in a new folder create: docker-compose.yaml

- i name the git branches after what i am working on, so i create a branch for this creation:

```sh
git add . 
git commit -m "created docker-compose file"
git checkout -b create-docker-compose
```

..if you follow the step on github then it would be easier when i do it that way.

- inside the docker-compose we now need to define the network of services:

```sh
services:
    # nginx
    web:
        image: nginx:latest
	ports: 
	    - "80:80"
    # php
    # mysql 
    # redis
```

- starting with the webserver nginx, getting an image from the docker hub (the official registry) and defining a mapping of the ports (on the left the port 80 on the local machine and on the right side of the colon the port on the container...with that we have a kind of gateway where we can look into the container from our local machine. 
Which will make a lot more sense in a second when run this network of services (though its just only one service in there, its a network of service nonetheless):

```sh
docker compose up
```

- behind the scenes it is reaching out to the docker registry, getting that nginx image, extracting it, and pulling it down to my computer. You will only have to do this once, because once you got that image on your computer, the next time you run docker compose up it will jst simply go and get the image straight from your computer where it is located , instead of pulling it from the docker hub again.\
All you need to do now is to visit: localhost:80 (which is the default port in the browser, so you can omit the :80), to check if the web server is successfully installed and working, in the browser you should have a "Welcome to nginx" screen.

To inspect and see what is running:

1. which container are running:
```sh 
docker ps
```

with that you should see a table with the columns container id, image, command, created, status, ports and names..and a table row for our nginx web server.

2. which services are running:
```sh
docker compose ps
```

- or you can look into docker desktop and see there the container, the pulled down image,..


The two main files in Docker are
- the docker-compose (for the creation of your app as a whole)
- and the Dockerfile (for creating the images, which are used to build the containers)

In the docker-compose.yaml you define the next service: 
- the application, which is going to be a php application, but we dont name it after the technology which is powering this, instead im going to name it after a descriptive name as for what it does and this is going to represent the app and all the application files will live inside of this container. So im simply going to call this one "app".

And we could do what we did previously where we went and got a pre-configured image and jst use that!\
Instead i am going to get a base PHP image from the hub but i want to customize it, i want to add my own extensions to it for example, like i want to use composer to install dependencies using it, so what i am going to do here is Im going to build my own image based on a base PHP image.\
In order to do that, instead of specifying the image that we're going to use, im going to actually build it locally and so im going to add a "build" key.\
In order to build it i need to give Docker some information: 
To build images, Docker needs to read something called a **Dockerfile**, so you need to tell Docker where that Dockerfile can be located. 
- so create a "php" folder (specifically for PHP Docker stuff such as config and Dockerfiles)
- and inside create a file called Dockerfile (to follow the convention, but sometimes you see: dockerfile.dev or dockerfile.prod)
- and in the docker-compose you tell docker where its located: ./php/Dockerfile

Then for the base image visit:
- hub.docker.com
- and search for php images
- click the "Tags" tag and look for a php 8.1 tag
- also i want fpm because im using nginx (more on that when i get to the nginx configuration): "sort by" -> "8.1-fpm"
- the one i choose is the "8.1-fpm-alpine"
- "alpine" basically means that its a minimalistic image with a minimal amount of extra software installed 

Back in the Dockerfile you can run a bunch of commands..
- first one is "FROM" and that means: from which image 
- the image we are looking for is: "php:8.1-fpm-alpine"
(consisting of the vendor php, and the actual image or rather the image tag)

- now you can run some other commands in order to install software, add extensions for things like composer and pulling dependencies etc.

- for the time being i just want to add a PHP extension so that i can use MySQL. The command you use in order to run commands is simply "RUN", and as im using a PHP base image, it means i have access to this command:\
"docker-php-ext-install"\
..which as the name might suggest is used for installing PHP extensions and what i want particularly is: "pdo pdo_mysql"

Let me recap the past steps regarding the docker-compose. When we last run the "docker compose up" what happened was:
- it looked at the image and built the container from that 
- whats going to happen here is its actually gonna look at the fact that we want to build the image first and its going to build the image from the Dockerfile when we run the cmd "docker compose up"
- but what i want to show first is how you can actually build the image in isolation by using following command:

```sh
docker build -t repo_name:php81 -f php/Dockerfile . 
```

- option: "-t" for giving the image a name 
- option "-f" means file and the path to the Dockerfile
- and the trailing dot means that we're building it from this folder location and so any resources which i wanted in my Dockerfile (even though ive not actually specified any) Docker would actually go and look for those inside the actual folder where im calling this command from.

If you now run:

```sh
docker images 
```

..you should see a new row for the newly created image

But: Just to be clear on sth, we didnt actually have to build that image separately. That will happen when we run docker compose, however i thought it'd be a good idea to show this step.

---

In the browser you already have seen the welcome-page of nginx, which we can see because this static html is laying on the server.
What we want to do is change our configuration to actually point to a location on the server where we're going to start putting our PHP files.

First lets have a look at the original config file and then you will be able to mentally make sense of all this.

```sh
docker compose up -d
```

- first step is to spin up the network again 
- "-d" means it will do it in detached mode, which basically means it ll hand the terminal back to me rather than showing whats happening inside the container. 

- to look inside the container you need the actual name of the container, so you could do:
```sh
docker compose ps
```

..and you should have two containers displayed: 
- "docker-php-app-1"
- "docker-php-web-1"

- and when you now type:
```sh
docker exec -it docker-php-web-1 sh
```

..you gonna execute a command on your container 
- "-it" means interactive so that you can interact with the container
- and the "t" means that i can do this in a terminal, and you are given a prompt
- then the name of the container
- "sh" is the command that we are executing (for "shell"..so you can shell into the container and run commands inside there) 

After executing this command you should have a prompt with a leading "#"

- lets just print out the content of the existing config file that has been delievered with nginx (open it in the console for example with cat):

```sh
cat /etc/nginx/conf.d/default.conf
```

Rest of my notes in: notes.txt!










