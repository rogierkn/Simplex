# Simplex

Easily deploy your web applications with these docker containers.  
Works great in combination with [Deployer](https://deployer.org) due to the `builder` container that can install your composer dependencies and install & compile your front-end assets through yarn (or npm).  

Simplex features:
- nginx - easily configurable by editing the supplied `nginx.conf`
- builder - install your dependencies & build your assets, accessible through SSH

### Prerequisites

**note**: I assume you use [Deployer](https://deployer.org) to deploy your application

Preferably you have 2 pairs of public/private keys. These will be used for:
- Connecting to the `builder` container from outside the host
- Accessing your git repositories




### Installing

Access the machine on which you want these containers to run and clone**Simplex**.  

Duplicate the `.env.example` to `.env` and open it

Set the path to your public key that will be used for SSH connections to the `builder` container    
`builder_ssh_key=~/.ssh/deployer.pub`  

Set the path to your private key that will be used to access your repositories  
`git_deploy_key=~/.ssh/gitdeploy`

Set the port you want to use to access the `builder` container  
`builder_ssh_port=50022`

Set the password for the `root` account in `database` container  
`DB_ROOT_PASSWORD=AVerySecretPassword` 
&nbsp;  

A full config would look like
```.dotenv
builder_ssh_key=~/.ssh/deployer.pub
git_deploy_key=~/.ssh/gitdeploy
builder_ssh_port=50022
DB_ROOT_PASSWORD=AVerySecretPassword
```

##### Configuring nginx

The default nginx config should be edited before your application will work
```
server {
    listen 80;
```
Replace awesome.app with your domain
```
    server_name awesome.app;
```
Replace index.php with your entry file. Symfony for example uses `app.php` by default
``` 
    index index.php;
```
The `/application` folder visible is mapped to the `applications` folder in `/var/www/`, this is where all your applications live. You should map the `root` to the document root of your application.
```
    root /var/www/applications/my-awesome-app/public;

    location / {
```
Here, also replace `index.php` with your entry file   
```
        try_files $uri /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
```
Here, also replace `index.php` with your entry file   
```
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

The only thing missing now is the actual application.  
With the above configuration you would clone your application repository into the `./applications/` folder.


Now run `docker-compose up -d` and wait for Docker to build the containers  
Once finished, you can access the builder container from outside with `ssh root@ip_address_of_host -i /path/to/private/key`

## Deployer

Deployer synergizes well with Simplex.
An excerpt of a deploy.php that can be used with Simplex.
```php
set('ssh_multiplexing', true);
set('git_tty', true); // necessary to add github to known hosts  
  
// Project name
set('application', "MyAwesomeApplication");  
  
// Project repository
set('repository', "git@github.com:johndoe/my-awesome-application");
  
host('ip_address_of_host')
	->stage('production')
	->user('root')
	->identityFile('~/.ssh/deployer') // The private key of the builder_ssh_key used earlier
	->roles('app')
	->port(50022) // The port defined earlier for the builder container
	->set('deploy_path', '/var/www/applications/my-awesome-application');
  
// Install composer dependencies and install & compile front-end assets
task('assets:build', '
	composer install --prefer-dist --no-interaction --ignore-platform-reqs
	yarn
	yarn prod
');
  
task('deploy', [
	'deploy:prepare',
	'deploy:lock',
	'deploy:release',
	'deploy:update_code',
    'assets:build',   
    'deploy:shared',
    'deploy:writable',
    'deploy:symlink',
    'deploy:unlock',
    'cleanup',
    'success'
]);
```
###### note  
Using Deployer would require also that you edit the document root used in your `nginx.conf`.  
It would be necessary to edit  
```root /var/www/applications/my-awesome-app/public;```  
into  
```root /var/www/applications/my-awesome-app/current/public;```  
This is because Deployer keeps multiple releases/deployments and symlinks the `/current/` folder to the latest release.
 

## License

MIT License - [LICENSE.md](LICENSE.md)
