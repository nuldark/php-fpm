# PHP FPM Docker Container Images

![GitHub release (with filter)](https://img.shields.io/github/v/release/nuldark/php-fpm)

## Supported tags and respective Dockerfile links
- [`8.3`, `8.3.4`, `latest`](https://github.com/nuldark/php/blob/master/8.3/Dockerfile)
- [`8.2`, `8.2.17`](https://github.com/nuldark/php/blob/master/8.2/Dockerfile)

## Quick reference
- **Image based on**:   
  [ghrc.io/nuldark/alpine](https://github.com/nuldark/php)

- **Supported architectures**:    
  `linux/amd64`, `linux/arm64`

- **Maintained by**:  
  [nuldark](https://github.com/nuldark)

- **Where to file issues**:    
  [https://github.com/nuldark/php-fpm/issues](https://github.com/nuldark/php/issues?q=)

## How to use this image

### start a php instance

```console
$ docker run --name some-php-fpm -d php-fpm
```

### ... via [`docker-compose`](https://github.com/docker/compose)
Example `docker-compose.yml` for `php-fpm`:

```yaml
version: '3.1'

services:
    php-fpm:
        image: nuldark/php-fpm:latest
        restart: always
        ports:
            - "9000:9000"
```

## Environment Variables

The php-fpm image uses several environment variables which are easy to miss.

| Variable      | Default Value | Description |
|---------------|---------------|-------------|
| `PHPIZE_DEPS` |               |             |

## LICENSE

View [license](https://www.php.net/license/) information for the software contained in this image.
