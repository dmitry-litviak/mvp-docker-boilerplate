# Docker for web development

This repository based on a [tutorial series](https://tech.osteel.me/posts/docker-for-local-web-development-why-should-you-care "Docker for local web development, introduction: why should you care?") about leveraging Docker for local web development.

Disclaimer:
I added few things, that I need for my development workflow including renaming some stuff and adding additional functionaly e.g. support for prod docker-compose. I know, that it is not recommended way to use docker-compose on production, but for small 1 VPS projects or MVPs I think it is a good way to go. If project becomes successfull you can always switch to k8s or serverless.

Also, it is assumed that this setup will be used mostly for Laravel as backend and Vue/React/Angualar/anything as frontend development. But you can apply needed changes to use it as you want both in monorepo or distributed way.

Later, it is possible, that additional Dockerfiles and configs will be added to support other frameworks.

## Local web development

### Content

This branch contains a [three-tier architecture](https://www.techopedia.com/definition/24649/three-tier-architecture) application running on a LEMP stack, supported by Docker and orchestrated by Docker Compose.

It includes:

* A container for Nginx;
* A container for the backend application (based on [Laravel](https://laravel.com/));
* A container for the frontend application (based on [Vue.js](https://vuejs.org/));
* A container for the worker;
* A container for MySQL;
* A container for Redis;
* A container for phpMyAdmin;
* A container for Ngrok;
* A container for Ofelia;
* A volume to persist MySQL data;
* A volume to persist Redis data.

The containers are based on Alpine images when available, for an optimised size.

## Prerequisites

Make sure [Docker Desktop for Mac or PC](https://www.docker.com/products/docker-desktop) is installed and running, or head [over here](https://docs.docker.com/install/) if you are a Linux user. You will also need a terminal running [Git](https://git-scm.com/) and [Bash](https://www.gnu.org/software/bash/).

This setup also uses localhost's ports 80 and 443, so make sure those are available.

### Directions of use

Add the following domains to your machine's `hosts` file:

```
127.0.0.1 backend.<name_of_project>.test frontend.<name_of_project>.test phpmyadmin.test
```

Clone the repository:

```
$ git clone git@github.com:dmitry-litviak/mvp-docker-boilerplate.git <name_of_project> && cd <name_of_project>
```

Remove .git folder to avoid errors
```
$ rm -rf .git
```

Run replacement command, that will replace all `<name_of_project>` in the project to your project name. Replace `<your_new_name>` in command with the name of your project.

OS X:
```
$ LC_ALL=C find ./ -type f -exec sed -i '' -e 's/<name_of_project>/<your_new_name>/g' {} \;
```

Linux:
```
$ find ./ -type f -exec sed -i 's/<name_of_project>/<your_new_name>/g' {} \;
```

For simplicity you can add the following function to your Bash start-up file (`.bashrc`, `.zshrc`...):
```
function <name_of_project>_mvp {
    cd <PATH>/<name_of_project> && bash mvp $*
        cd -
}
```

Where `<PATH>` is the absolute path leading to the folder where the repository was cloned.

Open a new terminal window or `source` your Bash start-up file for the changes to take effect, then run the following command:

```
$ <name_of_project>_mvp dev init
```

OR if didn't add anything to Bash start-up

```
$ cd <PATH>/<name_of_project>
$ mvp dev init
```

This will download and build the images listed in `docker-compose-dev.yml`, create and start the corresponding containers, and take other various steps to set up the project (this might take a while).

You're also likely to be prompted for your system account's password to install the SSL/TLS self-signed certificate, unless you're on Windows, in which case you'll need to install it [manually](https://www.thewindowsclub.com/manage-trusted-root-certificates-windows) (you will find it under `.docker/nginx/certs`).

Once the script is done, you can visit [frontend.<name_of_project>.test](https://frontend.<name_of_project>.test) and [backend.<name_of_project>.test](https://backend.<name_of_project>.test).

Learn about the available commands by displaying the menu:

```
$ mvp
```

#### Ngrok

To use Ngrok in order to expose the backend container to the Internet, you will first need to [create a free account](https://dashboard.ngrok.com/signup), and then update the `.env` file at the root of the project to fill in the [auth token](https://dashboard.ngrok.com/auth/your-authtoken). Once that's done, restart Ngrok's container and you'll be able to access Ngrok's interface at [localhost:4040](http://localhost:4040):

```
$ mvp dev restart ngrok
```

## Explanation

The images used by the setup are listed and configured in [`docker-compose-dev.yml`](https://github.com/dmitry-litviak/mvp-docker-boilerplate/blob/master/docker-compose-dev.yml).

When building and starting the containers based on the images for the first time, a MySQL database named `DB_NAME` as in your .env file is automatically created (you can pick a different name in the MySQL service's description in `.env`).

Minimalist Nginx configurations for the [backend application](https://github.com/dmitry-litviak/mvp-docker-boilerplate/blob/master/.docker/nginx/conf.d/backend.conf), the [frontend application](https://github.com/dmitry-litviak/mvp-docker-boilerplate/blob/master/.docker/nginx/conf.d/frontend.conf) and [phpMyAdmin](https://github.com/dmitry-litviak/mvp-docker-boilerplate/blob/master/.docker/nginx/conf.d/phpmyadmin.conf) are also copied over to Nginx's container, making them available at [backend.<name_of_project>.test](https://backend.<name_of_project>.test), [frontend.<name_of_project>.test](https://frontend.<name_of_project>.test) and [phpmyadmin.test](https://phpmyadmin.test) respectively (the database credentials are specified at `.env`).

The directories containing the backend and frontend applications are mounted onto both Nginx's and the applications' containers, meaning any update to the code is immediately available upon refreshing the page, without having to rebuild any container.

Moreover, the Vue.js development server is automatically started, meaning you can update the code and benefit from _hot-reload_ straight away.

The frontend application is consuming a simple endpoint from the backend application to fetch and display the text below the animated gif.

The database data is persisted in its own local directory through the volume `mysqldata`, which is mounted onto MySQL's container.

The same goes for the data stored into Redis, which is persisted in its own local directory through the volume `redisdata`, mounted onto Redis' container.

When running `mvp dev init`, all of the required steps to set up the project (installing dependencies, running database migrations, generating `.env` files, etc.) are automatically handled by a Bash function. Since the backend requires some extra steps, a [dedicated script](https://github.com/dmitry-litviak/mvp-docker-boilerplate/blob/master/.docker/backend/) is mounted onto and run directly in its container. At this folder you can find different scripts for different backends.

The SSL/TLS certificate is generated using OpenSSL on the Nginx container, and is installed automatically on your local machine unless you are on [Windows](https://www.thewindowsclub.com/manage-trusted-root-certificates-windows). It is also installed on the backend container, which allows it to directly communicate with the frontend container via the `frontend.demo.test` network alias defined for the Nginx service in `docker-compose.yml`.

Both the worker and backend services are based on the backend's Dockerfiles, in two separate stages. Example dockerfiles can be found at [dockerfiles-examples](https://github.com/dmitry-litviak/mvp-docker-boilerplate/blob/master/.docker/dockerfile-examples/). The worker is automatically started and will process any queued jobs.

Finally, scheduled tasks are handled by [Ofelia](https://hub.docker.com/r/mcuadros/ofelia), a job scheduler for Docker environments. The tasks are listed in the [config.ini](https://github.com/dmitry-litviak/mvp-docker-boilerplate/blob/master/.docker/scheduler/config.ini) file, which is mounted onto the scheduler's container. One task is already configured – Laravel's [scheduler](https://laravel.com/docs/scheduling), meaning you don't need to set up a cron entry.

Please refer to the [full series](https://tech.osteel.me/posts/docker-for-local-web-development-introduction-why-should-you-care "Docker for local web development, introduction: why should you care?") for a detailed explanation.

## Cleaning up

To stop the containers:

```
$ mvp dev stop
```

To destroy the containers:

```
$ mvp dev down
```

To destroy the containers and the associated volumes:

```
$ mvp dev down -v
```

To remove everything, including images and orphans containers:

```
$ mvp dev destroy
```

## Production web development
