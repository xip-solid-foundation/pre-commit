# pre-commit

A pre-commit hook installer with basic testing


## Pre-requisite

1. Docker Compose, [install Docker Compose](https://docs.docker.com/compose/install/)
2. PHP docker image

You may use [Symfony-Docker](https://github.com/dunglas/symfony-docker) to download and install Docker for Symfony.

## Getting Started

### Grant composer Access to the Private Repository
In your GitHub account, click your account icon in the top-right corner, select Settings and Developer Settings. Then select Personal Access Tokens.

Generate a new access token with Full control of private repositories privileges. Copy the access token value, switch to the terminal of your local computer, and execute the following command:

```
composer config --global --auth github-oauth.github.com [token]
```
Replace [token] with the value of your GitHub personal access token.

### Configure Your Project's composer.json File

Add the following to your project's composer.json file:

```
{
    "extra": {
        "symfony": {
            "endpoint": [
                "https://api.github.com/repos/xip-solid-foundation/pre-commit-recipe/contents/index.json",
                "flex://defaults"
            ]
        }
    }
}
```


## Installation

```
composer require xip-solid-foundation/pre-commit
```

## Install the Recipes in Your Project

If your private bundles/packages have not yet been installed in your project, run the following command:

```
composer update
```

If the private bundles/packages have already been installed and you just want to install the new private recipes, run the following command:

```
composer recipes
```