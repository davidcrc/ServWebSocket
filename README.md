## Inicializar

    composer create-project --prefer-dist laravel/laravel ServWebSocket
    php artisan key:generate
    php artisan serve --port=8001

## Agregando Laravel WebSockets al nuevo proyecto Laravel

    https://beyondco.de/docs/laravel-websockets/getting-started/installation
    composer require beyondcode/laravel-websockets
    php artisan vendor:publish
        [option] BeyondCode

    php artisan migrate:install

## Configurando la seguridad

    // - En config/websockets.php:

    PUSHER_APP_ID=111
    PUSHER_APP_KEY=public-key-124555
    PUSHER_APP_SECRET=secret-key-12345
    PUSHER_APP_CLUSTER=

    php artisan websockets:serve

    - Panel de admin web-sockets:
        http://127.0.0.1:8001/laravel-websockets

## Produccion

    -clonar en una carpeta: https://github.com/JuanDMeGon/servidor-websockets-realtime-con-laravel

    sudo composer install --no-dev

    sudo chown -R www-data storage/
    sudo chown -R www-data bootstrap/cache/

    sudo cp .env.exaple .env

    // sudo touch database/database.sqlite

    -- Copiar el .env que ya teneos configurado

        APP_URL=https://midominio.com

    - Migrates:
        sudo php artisan migrate

    - websockets
        // sudo php artisan websockets:serve
        sudo apt install supervisor
        cd /etc/supervisor/
        cd conf.d
        sudo nano midominio.com
            [program:unNombre]
            command=/usr/bin/php /var/www/midominio_carpeta/artisan websockets:serve
            autostart=true
            autorestart=true

        sudo supervisorctl update
        sudo supervisorctl start unNombre

        -- Asegrar se de que ya este corriendo el command
        sudo supervisorctl status

    - Usando nginx:

        cd /etc/nginx/sites-available
        sudo nano midominio.com

            server {
                listen        80;
                listen        [::]:80;

                server_name midominio.com;

                ## Las conexiones al websocket van a /app
                location /app {
                    proxy_pass             http://127.0.0.1:6001;
                    proxy_read_timeout     60;
                    proxy_connect_timeout  60;
                    proxy_redirect         off;

                    # Allow the use of websockets
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection 'upgrade';
                    proxy_set_header Host $host;
                    proxy_cache_bypass $http_upgrade;
                }
            }

    - Test
        sudo nginx -t

    - Habillitando midominio.com
        sudo ln -s /etc/nginx/sites-available/midominio.com /etc/nginx/sites-enabled/

    - Recargamos nginx:
        sudo systemctl reload nginx.service

## En produccion manejando conexiones HTTP (mejora)

    - sudo nano /etc/nginx/sites-available/midominio.com:
        
        server {
            listen        80;
            listen        [::]:80;

            server_name midominio.com;
            
            root /var/www/midominio_carpeta/public;
            
            index index.php;

            ## Manjera cualquier peticion a cualquier otra ubicacion diferente de /app
            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            ## Como manejarse con PHP
            location ~ \.php$ {
                include snippets/fastcgi-php.conf;

                ## Maneja otras peticiones a PHP socket
                fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
            }

            location /app {
                proxy_pass             http://127.0.0.1:6001;
                proxy_read_timeout     60;
                proxy_connect_timeout  60;
                proxy_redirect         off;

                # Allow the use of websockets
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
            }
        }
    
    - Test
        sudo nginx -t

    - Recargamos nginx:
        sudo systemctl reload nginx.service

## Certificado de seguridad con Acme:

    sudo acme.sh --issue -d midominio.com -w /var/www/midominio_carpeta/public/ --force

    sudo mkdir /etc/nginx/midominio.com

    sudo acme.hs -d midominio.com --install-cert --key-file /etc/nginx/certs/midominio.com/key.pem --fullchain-file /etc/nginx/certs/midominio.com/fullchain.pem --ca-file /etc/nginx/certs/midominio.com/ca.pem --realoadcmd "systemctl force-reload nginx.service" --force

    - Verificar archivos creado : ll /etc/nginx/certs/midominio.com/

    - Solo las conexiones pasadas por https seran obtenidas , las http iran a otro lado
    sudo nano /etc/nginx/sites-available/midominio.com:
        server {
            ## listen        80;
            ## listen        [::]:80;

            listen      443 ssl;
            listen        [::]:443 ssl;
            
            server_name midominio.com;
            
            root /var/www/midominio_carpeta/public;
            
            index index.php;

            ssl_certificate /etc/nginx/certs/midominio.com/fullchain.pem;
            ssl_certificate_key /etc/nginx/certs/midominio.com/key.pem;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            .
            .
            .
        }
    
    - Test
        sudo nginx -t

    - Recargamos nginx:
        sudo systemctl reload nginx.service

    - Ingresamos a laravel (redirecciona al nginx cuando entras con http):
        https://midominio.com