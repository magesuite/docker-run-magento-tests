# MageSuite Docker Container For Running Magento Tests

[![](https://images.microbadger.com/badges/image/magesuite/run-tests:stable.svg)](https://microbadger.com/images/magesuite/run-tests:stable "Docker Image Badge") [![](https://images.microbadger.com/badges/version/magesuite/run-tests:stable.svg)](https://microbadger.com/images/magesuite/run-tests:stable "Docker Image Version Badge")


*Use `stable` or `-stable` suffixed docker tags (are built from `vX.Y` git tags), others may be unstable.*

Please see the docker hub repo page [magesuite/run-tests](https://hub.docker.com/r/magesuite/run-tests).

## Software

The `master` branch uses Maria DB 10.2, Elasticsearch 6 and PHP 7.2 which are the latest versions supported by Magento.

Other versions will be avaialble on appropriately named branches / docker tags.

See [all tags on Docker Hub](https://hub.docker.com/r/magesuite/run-tests/tags).

# Notable versions 

| Version                   | Docker tag                    | Branch        | PHP   | DB            | Elasticsearch |
| ------------------------- | ----------------------------- | :-----------: | ----: | ------------: | ------------: |
| *php71-es5-mariadb100*    | _php71-es5-mariadb100-stable_ | master        | 7.1.X | MariaDB 10.0  | 5.X           |
| *php71-es6-mariadb101*    | _php71-es6-mariadb101-stable_ | master        | 7.1.X | MariaDB 10.1  | 6.X           | 
| *php72-es6-mariadb101*    | _php72-es6-mariadb101-stable_ | master        | 7.3.X | MariaDB 10.1  | 6.X           |
| *php73-es6-mariadb101*    | _php71-es6-mariadb101-stable_ | master        | 7.1.X | MariaDB 10.1  | 6.X           |
| *php71-es6-mariadb102*    | _php71-es6-mariadb102-stable_ | master        | 7.1.X | MariaDB 10.2  | 6.X           |
| *php72-es6-mariadb102*    | _php72-es6-mariadb102-stable_ | master        | 7.2.X | MariaDB 10.2  | 6.X           |
| *php73-es6-mariadb102*    | _php73-es6-mariadb102-stable_ | master        | 7.3.X | MariaDB 10.2  | 6.X           |
| *php71-es7-mariadb103*    | _php71-es7-mariadb103-stable_ | master        | 7.1.X | MariaDB 10.3  | 7.X           |
| *php72-es7-mariadb103*    | _php72-es7-mariadb103-stable_ | master        | 7.2.X | MariaDB 10.3  | 7.X           |
| *php73-es7-mariadb104*    | _php73-es7-mariadb104-stable_ | master        | 7.3.X | MariaDB 10.4  | 7.X           |
| *php71-es5-mysql57*       | _php71-es5-mysql57-stable_    | mysql         | 7.1.X | MySQL 5.7     | 5.X           |
| *php71-es6-mysql57*       | _php71-es6-mysql57-stable_    | mysql         | 7.1.X | MySQL 5.7     | 6.X           |
| *php72-es6-mysql57*       | _php72-es6-mysql57-stable_    | mysql         | 7.2.X | MySQL 5.7     | 6.X           |
| *php73-es6-mysql57*       | _php73-es6-mysql57-stable_    | mysql         | 7.3.X | MySQL 5.7     | 6.X           |
| *php71-es7-mysql57*       | _php71-es7-mysql57-stable_    | mysql         | 7.1.X | MySQL 5.7     | 7.X           |
| *php72-es7-mysql57*       | _php72-es7-mysql57-stable_    | mysql         | 7.2.X | MySQL 5.7     | 7.X           |
| *php73-es7-mysql57*       | _php73-es7-mysql57-stable_    | mysql         | 7.3.X | MySQL 5.7     | 7.X           |
| *php73-es7-mysql80*       | _php73-es7-mysql80-stable_    | mysql         | 7.3.X | MySQL 8.0     | 7.X           |
 
_Note: The `exec` mount opt on `/tmp` is needed for elasticsearch because it maps mem to temp files._
_The image is big and there's not really a way around it since it has a lot of software and is based on CentOS (which we use for local dev / production deployments, so we want to keep everything else close)._

## Basic usage

```bash
# Start the container with all services
docker run \
    --rm \
    --detach \
    --name mgs-test \
    --tmpfs /tmp:rw,exec \
    --tmpfs /var/lib/mysql \
    --tmpfs /var/lib/elasticsearch \
    --tmpfs /var/www/html/generated \
    --tmpfs /var/www/html/var \
    --tmpfs /dev/tests/integration/tmp \
    --volume "$(pwd):/var/www/html" \
    "magesuite/run-tests:stable"

# Wait until healthcheck is green
while [[ "$(docker inspect --format='{{json .State.Health.Status}}' mgs-test)" == '"starting"' ]] ; do sleep 1s && echo "Waiting for start"; done
    
# Then later execute your test suite
docker exec \
    --tty \
    --user "$(id -u):$(id -g)" \
    mgs-test \
    /usr/bin/mgs-run-tests ci creativestyle
```

During the testing you can get into the bash shell to poke around:

```bash
docker exec -it mgs-test /bin/bash
```

## Run everthing at once

**Warning: The approach with switching to arbitrary UID/GID via sudo doesn't work everywhere, workaround is being prepared.**

As an alternative you could run everything at once as the 
tests will wait for healthcheck to become green before proceeding:

```bash
docker run \
    --tty \
    --tmpfs /tmp:rw,exec \
    --tmpfs /var/lib/mysql \
    --tmpfs /var/lib/elasticsearch \
    --tmpfs /var/www/html/generated \
    --tmpfs /var/www/html/var \
    --tmpfs /dev/tests/integration/tmp \
    --volume "$(pwd):/var/www/html" \
    "magesuite/run-tests:stable" \
    "/usr/bin/elasticsearch-server" \
    "/usr/bin/mysql-server" \
    "/usr/bin/mgs-run-tests ci creativestyle $(id -u) $(id -g)"
```

## Run a bash shell without starting any services

```bash
docker run \
    --tty \
    --interactive \
    --tmpfs /tmp:rw,exec \
    --tmpfs /var/lib/mysql \
    --tmpfs /var/lib/elasticsearch \
    --tmpfs /var/www/html/generated \
    --tmpfs /var/www/html/var \
    --tmpfs /dev/tests/integration/tmp \
    --volume "$(pwd):/var/www/html" \
    "magesuite/run-tests:stable" \
    "/bin/bash"
```

You can start the services while inside by doing:

```bash
/usr/bin/elasticsearch-server &
/usr/bin/mysql-server &
```
