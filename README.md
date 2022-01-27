# pre-commit

A pre-commit hook installer with basic testing


## Pre-requisite

1. Docker Compose, [install Docker Compose](https://docs.docker.com/compose/install/)
2. PHP docker image

You may use [Symfony-Docker](https://github.com/dunglas/symfony-docker) to download and install Docker for Symfony.

Installation
=

Applications that use Symfony Flex Installation
-

```
$ composer require xip-solid-foundation/pre-commit
```

Applications that don't use Symfony Flex
-
### Step 1: Install the Bundle

Open a command console, enter your project directory and execute the
following command to install the latest stable version of this bundle:

```console
$ composer require xip-solid-foundation/pre-commit
```

### Step 2: Added Pre-commit file


Example:
```#!/usr/bin/env bash
# Automatic usage by installing symbolic link:
# ln -sf ../../bin/pre-commit .git/hooks/pre-commit

# Enable TTY, so fastest runs on a single line and looks cooler
exec < /dev/tty

if [[ -z `docker ps -q --no-trunc | grep $(docker-compose ps -q apache2)` ]]; then
    echo -e "\e[41m COMMIT FAILED: Docker-compose is not running! \e[0m";
    exit 1
fi

# Install phpunit synchronously to avoid problems.
docker-compose exec php bin/phpunit install

docker-compose exec -T php bin/console cache:warmup --env=prod
if [[ $? != "0" ]]
then
    echo -e "\e[41m COMMIT FAILED: Cannot warm cache for env=prod! \e[0m":
    exit 1
fi

# see which files you wanted to commit, fix code style and re-add
FILES=` git status --porcelain | grep -e '^[AM]\(.*\).php$' | cut -c 3- | tr '\n' ' '`
if [[ $FILES ]]
then
    FILES_NO_APPLICATION=`echo ${FILES} | sed "s/api\///g"`
    docker-compose exec -T php vendor/bin/php-cs-fixer fix
    git add ${FILES}
fi

# remove outdated code coverage files
rm -rf var/coverage.txt public/test.html

# warmup test cache to speed up tests
docker-compose exec php bin/console cache:warmup --env=test

# Run Tests and generate code coverage
docker-compose exec php bash -c "XDEBUG_MODE=coverage vendor/bin/paratest --coverage-html=public/test.html --coverage-text=var/coverage.txt tests"
if [[ $? != "0" ]]
then
    echo -e "\e[41m COMMIT FAILED: You have test errors! \e[0m":
    exit 1
fi

# merge code coverage
echo "Full HTML coverage at http://0.0.0.0/test.html/index.html"

OK=`docker-compose exec -T php head -n10 var/coverage.txt | grep Lines | grep "100.00%"`
if [[ ${OK} = "" ]]
then
    echo -e "\e[41m COMMIT FAILED: Code coverage not 100%! \e[0m":
    exit 1
fi

echo ""
```

Usage
=
Now you can run the pre-commit by using 

```
$ bin/pre-commit
```