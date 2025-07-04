# Sail Runtime Image

![Ariaieboy Sail Runtime Image](https://preview.dragon-code.pro/Ariaieboy/Sail%20Runtime%20Image.svg?brand=docker&github%5Brepository%5D=ariaieboy%2Fsail-runtime-image)
[![Docker Badge](https://img.shields.io/docker/pulls/ariaieboy/sail-runtime-image)](https://hub.docker.com/r/ariaieboy/sail-runtime-image/)
[![build](https://github.com/ariaieboy/sail-runtime-image/actions/workflows/build.yml/badge.svg)](https://github.com/ariaieboy/sail-runtime-image/actions/workflows/build.yml)
[![build-alpine](https://github.com/ariaieboy/sail-runtime-image/actions/workflows/build-alpine.yml/badge.svg)](https://github.com/ariaieboy/sail-runtime-image/actions/workflows/build-alpine.yml)

One of the Laravel Sail problems is the build step. Depending on your network and machine power, it might take a while
to build the image locally.
You can remove the build step by using our Pre-built images.

## Available Tags

| Tags                        | Description                             |
|-----------------------------|-----------------------------------------|
| `8.1`,`8.1-20`              | PHP 8.1, Node 20                        |
| `8.1-22`                    | PHP 8.1, Node 22                        |
| `8.1-24`                    | PHP 8.1, Node 24                        |
| `8.2`,`8.2-20`              | PHP 8.2, Node 20                        |
| `8.2-22`                    | PHP 8.2, Node 22                        |
| `8.2-24`                    | PHP 8.2, Node 24                        |
| `8.3`,`8.3-20`              | PHP 8.3, Node 20                        |
| `8.3-22`                    | PHP 8.3, Node 22                        |
| `8.3-24`                    | PHP 8.3, Node 24                        |
| `8.4`,`8.4-22`              | PHP 8.4, Node 22                        |
| `8.4-24`                    | PHP 8.4, Node 24                        |
| `8.x-alpine`,`8.x-y-alpine` | same as above with Alpine as base image |

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

Then you need to change the image to `ariaieboy/sail-runtime-image:version` the version can be `8.1`,`8.2`,`8.3`:

```diff
services:
    laravel.test:
+       image: ariaieboy/sail-runtime-image:8.3
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
      SUPERVISOR_PHP_USER: '${SUPERVISOR_PHP_USER:-sail}'
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

### `custom`

You can provide a new env variable called `SUPERVISOR_PHP_COMMAND` and pass a custom supervisor php command.

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

To update the image you only need to run `sail pull` and restart your containers.

## Version Support Policy

Every Image has lots of dependencies that can have multiple versions.

We break this into 3 major components.

* OS version:

In base images, we are going to use the latest LTS version of the Ubuntu. For example, when PHP 8.3 was released the
latest LTS version of Ubuntu was 22.04 so we created the base image using the Ubuntu 22.04 and after releasing Ubuntu
24.04 we upgraded to the 24.04.
In alpine variants, we use the PHP8.x-cli-alpine image as the base so we are tied to the PHP official docker image for
the alpine version.

* Node.js version:

In new tags, we are only supporting the latest LTS version of the Node.js. For example, PHP 8.3 only supports Node.js
version >=20.
If a new LTS version of Node.js is released, we are going to add a new image tag with that specific Node.js version. But
the default one will only be the first version.
We also remove the old Node.js versions from the build process after they reach EOL.

* PostgreSQL Client:

Since every major version of PostgresSQL Client is backward compatible with all supported versions of PostgreSQL, we
will
update the Client to the latest version in all tags.

* Other components:

Every other component will be updated to the latest version with each build.

> We only support the active maintenance versions of the packages. For example, if the Node.js 18 support ends we are
> going to remove the tags that contain Node.js 18 from the build process.

> You can still pull the latest builds for EOL tags, but they will not be updated anymore, and you will need to maintain
> those tags yourself.

> We removed bunjs from the alpine image since it has some problems. use the default image instead of alpine variant if
> you use bunjs.

## License

Copyright © AriaieBOY

Sail Runtime Image is open-sourced software licensed under the [MIT license](LICENSE).

## Sync status

Synced with [laravel sail 1.43.1](https://github.com/laravel/sail/releases/tag/v1.43.1)