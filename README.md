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
