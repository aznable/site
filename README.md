# Aznable Site

Guide for creating web applications under [Aznable Infra](https://github.com/aznable/infra).

## Installation

Install `Aznable Site` by cloning its Github repository:

```
git clone https://github.com/aznable/site.git
cd site
cp .env.example .env
rm -rf .git
```

## Initial Configuration

Modify the values in the `.env` file:

```
###################################
# Let's Encrypt settings
###################################
LETSENCRYPT_MAIL=me@aznable.test
LETSENCRYPT_HOST=aznable.test

###################################
# NGINX Proxy settings
###################################
VIRTUAL_HOST=aznable.test
VIRTUAL_DEST=/
VIRTUAL_PATH=/
```

### Path-based routing

Full reference: [https://github.com/nginx-proxy/nginx-proxy#path-based-routing](https://github.com/nginx-proxy/nginx-proxy#path-based-routing)

The default configuration already includes `VIRTUAL_PATH` but does not cater `VIRTUAL_DEST`. `VIRTUAL_DEST` can only be used if requests are expected to not contain a sub-path and the generated links contain the sub-path.

```
# .env

###################################
# NGINX Proxy settings
###################################
VIRTUAL_HOST=aznable.test
VIRTUAL_DEST=/
VIRTUAL_PATH=/hello
```

``` yaml
# docker.yml

services:
  web:
    ...
    environment:
      LETSENCRYPT_HOST: ${LETSENCRYPT_HOST}
      LETSENCRYPT_EMAIL: ${LETSENCRYPT_MAIL}
      VIRTUAL_HOST: ${VIRTUAL_HOST}
      VIRTUAL_PATH: ${VIRTUAL_PATH}
      VIRTUAL_DEST: ${VIRTUAL_DEST} # add another variable
    ...
```

In the example above, requesting `aznable.test/hello` in the URL redirects to the specified container.

#### Special routing inside applications

If the container contains special routes in its own (e.g. an PHP application), it will be automatically be routed from the specified URL:

```
hello/
├─ web/
│  ├─ index.php
├─ .env
├─ docker.yml
```

``` php
<?php

// web/index.php of the first container
// NOTE: This assumes that "VIRTUAL_DEST" is included in docker.yml

$root = realpath(dirname(__DIR__));

require $root . '/vendor/autoload.php';

// Initializes the router application
$app = new Zapheus\Coordinator;

// Creates a HTTP route of GET /
$app->get('/', function ()
{
    return 'Hello world from Zapheus!';
});

$app->get('/test', function ()
{
    return 'This is a test route!';
});

// Handles the server request
echo $app->run();
```

Based from the above example, requesting `aznable.test/hello/test` will return `This is a test route!` in the web browser. Requesting `aznable.test/hello` only also returns `Hello world from Zapheus!`.