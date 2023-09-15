# Sail Runtime Image

![Ariaieboy Sail Runtime Image](https://preview.dragon-code.pro/Ariaieboy/Sail%20Runtime%20Image.svg?brand=docker&github%5Brepository%5D=ariaieboy%2Fsail-runtime-image)
One of the Laravel Sail problems is the build step. Depending on your network and machine power, it might take a while
to build the image locally.
You can remove the build step by using our Pre-built images.

## Available Tags

| Tags                        | Description                             | Postgres-client |
|-----------------------------|-----------------------------------------|-----------------|
| `8.0`,`8.0-20`              | PHP 8.0, Node 20                        | 15              |
| `8.0-18`                    | PHP 8.0, Node 18                        | 15              |
| `8.1`,`8.1-20`              | PHP 8.1, Node 20                        | 15              |
| `8.1-18`                    | PHP 8.1, Node 18                        | 15              |
| `8.2`,`8.2-20`              | PHP 8.2, Node 20                        | 15              |
| `8.2-18`                    | PHP 8.2, Node 18                        | 15              |
| `8.3`,`8.3-20`              | PHP 8.3, Node 20                        | 16              |
| `8.3-18`                    | PHP 8.3, Node 18                        | 16              |
| `8.x-alpine`,`8.x-y-alpine` | same as above with Alpine as base image | 15              |

## Usage

After installing [laravel-sail](https://laravel.com/docs/sail) you must remove the build step:

```diff
services:
    laravel.test:
-        build:
-            context: ./vendor/laravel/sail/runtimes/8.2
-            dockerfile: Dockerfile
-            args:
-                WWWGROUP: '${WWWGROUP}'
        image: sail-8.2/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
```

Then you need to change the image to `ariaieboy/sail-runtime-image:version` the version can be `7.4`,`8.0`,`8.1`,`8.2`:

```diff
services:
    laravel.test:
+       image: ariaieboy/sail-runtime-image:8.2
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
+           WEBSERVER: cli
+           WWWGROUP: '${WWWGROUP}'
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
```

The final `laravel.test` service should look like this:

```yml
services:
  laravel.test:
    image: ariaieboy/sail-runtime-image:8.2
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    ports:
      - '${APP_PORT:-80}:80'
      - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
    environment:
      WEBSERVER: cli
      WWWUSER: '${WWWUSER}'
      WWWGROUP: '${WWWGROUP}'
      LARAVEL_SAIL: 1
      XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
      XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
    volumes:
      - '.:/var/www/html'
    networks:
      - sail
```

For using Alpine you can add `user` to the docker-compose service:

```yml
services:
  laravel.test:
    image: ariaieboy/sail-runtime-image:8.2-alpine
    user: "${WWWUSER}:${WWWGROUP}"
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    ports:
      - '${APP_PORT:-80}:80'
      - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
    environment:
      WEBSERVER: cli
      LARAVEL_SAIL: 1
      XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
      XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
    volumes:
      - '.:/var/www/html'
    networks:
      - sail
```

## Customize Webserver

Our images support 3 Separate web server that you can choose from by setting the environment value of `WEBSERVER` in
your laravel service in the `docker-compose` file:

### `cli` (default)

This option uses the default `php artisan serve` supervisor conf from the `laravel-sail` repo.

### `octane`

This option uses `php artisan octane:serve` and will serve your application using octane instead of the
built-in `php artisan serve`

### `octane-watch`

This option uses octane webserver with `--watch` option.

## Difference with laravel-sail runtimes

### Pros

* It's Pre-built and you only need to pull it from docker hub.
* It supports 3 different Webserver without needing to change the image and rebuild it.
* Additional packages installed including `libavif-bin pnpm svgo libheif-dev jpegoptim optipng pngquant gifsicle`
* It will update weekly at 00:00 on Wednesday
* The image size is smaller than Laravel Sail default build
* Our Alpine variant is 60% smaller (475 vs 180 MB)

### cons

* It's not as customizable as `laravel-sail` if you want to customize it you need to build another image based on our
  images and build that image.

## Update the image

For updating the image you only need to run `sail pull` and restart your containers.

## License

Copyright © AriaieBOY

Sail Runtime Image is open-sourced software licensed under the [MIT license](LICENSE).
