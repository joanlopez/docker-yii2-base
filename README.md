# docker-yii2-base
Yii2 base image for dockerized Yii2 projects

## Supported tags and respective `Dockerfile` links

- [`7.2.2-apache`, `latest` (*apache/7.2.2/Dockerfile*)](https://github.com/joanlopez/docker-yii2-base/blob/master/apache/7.2.2/Dockerfile)
- [`7.1.15-apache` (*apache/7.1.15/Dockerfile*)](https://github.com/joanlopez/docker-yii2-base/blob/master/apache/7.1.15/Dockerfile)

Yii 2 Base
==========

This is a base image for Yii 2 projects.

> **IMPORTANT: The image does *not* contain an app template!**
> So you must always first combine it with your own application code to make it work!

The main purpose of this image is,

 * to provide a PHP runtime environment that is configured for Yii and
 * that has the base yii2 composer packages pre-installed.

## 1. Available versions

There is only one available flavour of this image

 * **Apache with PHP module** (based on `php:7.2.2-apache`)
 
## 2. How to use this Image

### 2.1 Using your own Application Template

If you don't want to use that base template you can still build an application
from scratch. But still we recommend to study that template first.

Before you build your own application template, you should understand the basic
idea behind this image:

 * The image has all yii2 related composer packages pre-installed under `/var/www/vendor`.
 * The application code is expected under `/var/www/html`, with
   the public directory being `/var/www/html/web`.
 * You will *never* install any composer packages locally, but
   always into your container. When you do so, it will either override
   or add more packages to those already contained in this image.

You'll also need to prepare some application code locally which will be copied to your
image during the build phase. To start, you could use the official base image:

```
composer create-project --no-install yiisoft/yii2-app-basic
```

> **Note:** You need to fix the paths to `autoload.php` and `Yii.php` in the
> `index.php` file and also add a `'vendorPath' => '/var/www/vendor'` option
> in the `config/web.php`.


#### 2.2.1 Using the Apache Variant

Create a `Dockerfile` in your application directory:

```
FROM joanlopez/yii2-base:latest

# Copy your app's source code into the container
COPY . /var/www/html
```

and a `docker-compose.yml`:

```
web:
    build: ./
    ports:
        - "8080:80"
    expose:
        - "80"
    volumes:
        - ./:/var/www/html/
```

Now you're ready to run `docker-compose up` to start your app. It should
be available from `http://localhost:8080` or your boot2docker VM if you use that.


### 2.3 Adding Composer Packages

To add composer packages, you need to provide a `composer.json` with
some modifications:

```
{
  "require": {
    "php": ">=5.4.0",
    "yiisoft/yii2": "2.0.13.1",
    "yiisoft/yii2-bootstrap": "~2.0.0",
    "yiisoft/yii2-jui": "~2.0.0",
    "yiisoft/yii2-swiftmailer": "~2.0.0"
  },
  "require-dev": {
    "yiisoft/yii2-debug": "~2.0.0",
    "yiisoft/yii2-gii": "~2.0.0",
    "yiisoft/yii2-codeception": "~2.0.0",
    "yiisoft/yii2-faker": "~2.0.0"
  },
  "config": {
    "process-timeout": 1800,
    "vendor-dir": "/var/www/vendor"
  },
  "extra": {
    "asset-installer-paths": {
      "npm-asset-library": "../vendor/npm",
      "bower-asset-library": "../vendor/bower"
    }
  }
}
```

Note the `vendor-dir` configuration, which is crucial for this setup. It's also
important, that the versions there match those of this image. Otherwhise you again
loose the advantage of reusing the composer packages contained in the image.

You also have to map the local directory into the container in your `docker-compose.yml`.
If you have problems with githubs rate limit, you can provide a github API token.

```
web:
    build: ./
    ports:
        - "8080:80"
    expose:
        - "80"
    volumes:
        - ./:/var/www/html/
    environment:
        API_TOKEN: "<YOUR GITHUB API TOKEN>"
```

Now you can run the bundled `composer` command in your container.

```
docker-compose run --rm web compose update myrepo/mypackage
```

### 2.4 Adding PHP Extensions

As the `apache` flavour extend from the official [php](https://registry.hub.docker.com/u/library/php/)
image, you can use `docker-php-ext-install` in your `Dockerfile`. You may also have to install
some required packages with `apt-get install` first. Here's an example:

```
RUN apt-get update \
    && apt-get -y install \
            libfreetype6-dev \
            libjpeg62-turbo-dev \
            libmcrypt-dev \
            libpng12-dev \
        --no-install-recommends \
    && rm -r /var/lib/apt/lists/* \
    && docker-php-ext-install iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd
```
