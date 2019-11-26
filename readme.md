# Docker Laravel

### Step 1 

Get current laravel installation use the <em>period</em> to install laravel in current directory, if you need to install laravel make sure you hav ethe following file structure

Procfile
docker/             
docker-compose.yml 
heroku-config.py   
readme.md

```bash

$ composer create-project --prefer-dist laravel/laravel .

```


### Step 2
Next, use Docker's composer image to mount the directories that you will need for your Laravel project and avoid the overhead of installing Composer globally:

```bash
$ docker run --rm -v $(pwd):/app composer install

```

### Step 3

As a final step, set permissions on the project directory(I dont expererience this issue before you keep moving along check the permission) so that it is owned by your non-root user, move one level up:

```bash
cd ../
$ sudo chown -R $USER:$USER docker-laravel

```

### Step 4

#### Running the Containers and Modifying Environment Settings

Now that you have defined all of your services in your docker-compose file and created the configuration files for these services, you can start the containers. As a final step, though, we will make a copy of the .env.example file that Laravel includes by default and name the copy .env, which is the file Laravel expects to define its environment:
```bash
$ cp .env.example .env

```
We will configure the specific details of our setup in this file once we have started the containers.

### Step 5

With all of your services defined in your docker-compose file, you just need to issue a single command to start all of the containers, create the volumes, and set up and connect the networks:
```bash
$ docker-compose up -d
```

When you run docker-compose up for the first time, it will download all of the necessary Docker images, which might take a while. Once the images are downloaded and stored in your local machine, Compose will create your containers. The -d flag daemonizes the process, running your containers in the background.

Once the process is complete, use the following command to list all of the running containers:

```bash
$ docker ps

```

You will see the following output with details about your `app`, `webserver`, and `db` containers:

``` bash
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                      NAMES
bc8f26d43109        digitalocean.com/php   "docker-php-entrypoi…"   14 seconds ago      Up 13 seconds       9000/tcp                                   app
481a60d5e1f2        mariadb:latest         "docker-entrypoint.s…"   14 seconds ago      Up 13 seconds       0.0.0.0:3306->3306/tcp                     db
1bef4a1e8330        nginx:alpine           "nginx -g 'daemon of…"   14 seconds ago      Up 13 seconds       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   webserver
```

### Step 6
The CONTAINER ID in this output is a unique identifier for each container, while NAMES lists the service name associated with each. You can use both of these identifiers to access the containers. IMAGE defines the image name for each container, while STATUS provides information about the container's state: whether it's running, restarting, or stopped.

You can now modify the .env file on the app container to include specific details about your setup.

Open the file using docker-compose exec, which allows you to run specific commands in containers. In this case, you are opening the file for editing:

```bash
$ docker-compose exec web nano .env

```

```bash
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laraveluser
DB_PASSWORD=secret

```

### Step 7

Next, set the application key for the Laravel application with the php artisan key:generate command. This command will generate a key and copy it to your .env file, ensuring that your user sessions and encrypted data remain secure:
```bash
$ docker-compose exec web php artisan key:generate

```

### Step 8

You now have the environment settings required to run your application. To cache these settings into a file, which will boost your application's load speed, run:

```bash
$ docker-compose exec web php artisan config:cache

```
### Step 8a

run the yarn in the disposable container

```bash
$ docker run --rm -it -v $(pwd):/app -w /app node yarn

$ docker run --rm -it -v $(pwd):/app -w /app node yarn dev
```

Your configuration settings will be loaded into /var/www/bootstrap/cache/config.php on the container.

As a final step, visit `http://localhost` in the browser. You will see the following home page for your Laravel application:

With your containers running and your configuration information in place, you can move on to configuring your user information for the laravel database on the db container.

<p align="center"><img src="https://assets.digitalocean.com/articles/laravel_docker/laravel_home.png"></p>

With your containers running and your configuration information in place, you can move on to configuring your user information for the laravel `database` on the db container.

### Step 9 — Creating a User for MySQL

To create a new user, execute an interactive bash shell on the db container with `docker-compose` exec:

```bash
$ docker-compose exec db bash
```
Inside the container, log into the MySQL root administrative account:

```bash
$ root@481a60d5e1f2:/# mysql -u root -p 
```

Start by checking for the database called laravel, which you defined in your docker-compose file. Run the show databases command to check for existing databases:

```bash
$ MariaDB [(none)]> show databases;

```

```bash
+--------------------+
| Database           |
+--------------------+
| information_schema |
| laravel            |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.001 sec)
```
Next, create the user account that will be allowed to access this database. Our username will be `laraveluser` as well, though you can replace this with another name if you'd prefer. Just be sure that your username and password here match the details you set in your `.env` file in the previous step:

```bash
MariaDB [(none)]># GRANT ALL ON laravel.* TO 'laraveluser'@'%' IDENTIFIED BY 'secret';
```
Flush the privileges to notify the MySQL server of the changes:
```bash
MariaDB [(none)]># FLUSH PRIVILEGES;
```
Exit the DB
```bash
MariaDB [(none)]># EXIT;
```
Exit the container

```bash
root@481a60d5e1f2:/# exit
```

### Step 10
Step 10 — Migrating Data and Working with the Tinker Console

First, test the connection to MySQL by running the Laravel artisan migrate command, which creates a migrations table in the database from inside the container:

```bash
$ docker-compose exec web php artisan migrate
```
This command will migrate the default Laravel tables. The output confirming the migration will look like this:

```bash
#Output
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table
```

Once the migration is complete, you can run a query to check if you are properly connected to the database using the tinker command:

```bash
$ docker-compose exec web php artisan tinker
```
```bash
#Psy Shell v0.9.9 (PHP 7.2.14 — cli) by Justin Hileman
>>> \DB::table('migrations')->get();
```
You will see output that looks like this:

```bash
=> Illuminate\Support\Collection {#2899
     all: [
       {#2905
         +"id": 1,
         +"migration": "2014_10_12_000000_create_users_table",
         +"batch": 1,
       },
       {#2908
         +"id": 2,
         +"migration": "2014_10_12_100000_create_password_resets_table",
         +"batch": 1,
       },
     ],
   }
>>> 
```








<p align="center"><img src="https://laravel.com/assets/img/components/logo-laravel.svg"></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/d/total.svg" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/license.svg" alt="License"></a>
</p>

## About Laravel

Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel attempts to take the pain out of development by easing common tasks used in the majority of web projects, such as:

- [Simple, fast routing engine](https://laravel.com/docs/routing).
- [Powerful dependency injection container](https://laravel.com/docs/container).
- Multiple back-ends for [session](https://laravel.com/docs/session) and [cache](https://laravel.com/docs/cache) storage.
- Expressive, intuitive [database ORM](https://laravel.com/docs/eloquent).
- Database agnostic [schema migrations](https://laravel.com/docs/migrations).
- [Robust background job processing](https://laravel.com/docs/queues).
- [Real-time event broadcasting](https://laravel.com/docs/broadcasting).

Laravel is accessible, yet powerful, providing tools needed for large, robust applications.

## Learning Laravel

Laravel has the most extensive and thorough [documentation](https://laravel.com/docs) and video tutorial library of any modern web application framework, making it a breeze to get started learning the framework.

If you're not in the mood to read, [Laracasts](https://laracasts.com) contains over 1100 video tutorials on a range of topics including Laravel, modern PHP, unit testing, JavaScript, and more. Boost the skill level of yourself and your entire team by digging into our comprehensive video library.

## Laravel Sponsors

We would like to extend our thanks to the following sponsors for helping fund on-going Laravel development. If you are interested in becoming a sponsor, please visit the Laravel [Patreon page](https://patreon.com/taylorotwell):

- **[Vehikl](https://vehikl.com/)**
- **[Tighten Co.](https://tighten.co)**
- **[Kirschbaum Development Group](https://kirschbaumdevelopment.com)**
- **[64 Robots](https://64robots.com)**
- **[Cubet Techno Labs](https://cubettech.com)**
- **[Cyber-Duck](https://cyber-duck.co.uk)**
- **[British Software Development](https://www.britishsoftware.co)**
- **[Webdock, Fast VPS Hosting](https://www.webdock.io/en)**
- **[DevSquad](https://devsquad.com)**
- [UserInsights](https://userinsights.com)
- [Fragrantica](https://www.fragrantica.com)
- [SOFTonSOFA](https://softonsofa.com/)
- [User10](https://user10.com)
- [Soumettre.fr](https://soumettre.fr/)
- [CodeBrisk](https://codebrisk.com)
- [1Forge](https://1forge.com)
- [TECPRESSO](https://tecpresso.co.jp/)
- [Runtime Converter](http://runtimeconverter.com/)
- [WebL'Agence](https://weblagence.com/)
- [Invoice Ninja](https://www.invoiceninja.com)
- [iMi digital](https://www.imi-digital.de/)
- [Earthlink](https://www.earthlink.ro/)
- [Steadfast Collective](https://steadfastcollective.com/)
- [We Are The Robots Inc.](https://watr.mx/)
- [Understand.io](https://www.understand.io/)

## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
