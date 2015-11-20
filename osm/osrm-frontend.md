# Install OSRM-frontend on a server

We will use osrm-frontend-v2, the same proposed at : https://map.project-osrm.org.

This frontend is builds heavily ontop of Leaflet Routing Machine. It provide a simple OSRM integration in a webpage.

## Configure nginx

First, we configure nginx to create a new http server :
```
apt-get install nginx
```

And edit **/etc/nginx/sites-available/default** and put this :
```
server {
        listen          80;
        server_name     example.org  www.example.org;

        client_max_body_size 20M;

        index           index.html;
        root            /usr/share/nginx/www/osrm;

        location / {
                try_files $uri $uri/ /index.php;
        }
}
```
Custom this config to suit your needs (server name, port, ...)

Now we have an http server which serves **/usr/share/nginx/www/osrm** directory.

## Install OSRM-frontend

The OSRM-frontend install depends on NPM, so we install it :
```
curl -sL https://deb.nodesource.com/setup_4.x | bash -
apt-get install -y nodejs
```

We can start to install OSRM-frontend in the **/usr/share/nginx/www/osrm** directory : 
```
git clone https://github.com/Project-OSRM/osrm-frontend-v2.git /usr/share/nginx/www/osrm
cd /usr/share/nginx/www/osrm
npm install
make
```

Now you have an OSRM-frontend accessible on http://<server_ip>.

Actually it get data from the official OSRM project server, for use your own OSRM-backend server, just configure **/usr/share/nginx/www/osrm/src/leaflet_options.js** :
```
services: [{
    label: 'Car (fastest)',
    path: 'http://<your osrm-backend server>/viaroute'
  }],
```

And every other options you want, remember that to apply those modifs, you need to re-make OSRM-frontend :
```
cd /usr/share/nginx/www/osrm
make
```
