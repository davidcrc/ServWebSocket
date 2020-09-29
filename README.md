## Inicializar

    composer create-project --prefer-dist laravel/laravel ServWebSocket
    php artisan key:generate
    php artisan serve --port 8001

## Agregando Laravel WebSockets al nuevo proyecto Laravel

    composer require beyondcode/laravel-websockets
    php artisan vendor:publish
        [option] BeyondCode