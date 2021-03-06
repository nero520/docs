# What is Composer?

Composer is a tool for dependency management in PHP, written in PHP. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.

You can read more about Composer in our [official documentation](https://getcomposer.org/doc/).

%%LOGO%%

# How to use this image

### Basic usage

Running the `composer` image is as simple as follows:

```console
$ docker run --rm --interactive --tty \
  --volume $PWD:/app \
  %%IMAGE%% install
```

### Persistent cache / global configuration

You can bind mount the Composer home directory from your host to the container to enable a persistent cache or share global configuration:

```console
$ docker run --rm --interactive --tty \
  --volume $PWD:/app \
  --volume $COMPOSER_HOME:/tmp \
  %%IMAGE%% install
```

**Note:** this relies on the fact that the `COMPOSER_HOME` value is set to `/tmp` in the image by default.

Or if you are following the XDG specification:

```console
$ COMPOSER_HOME=$HOME/.config/composer \
  COMPOSER_CACHE_DIR=$HOME/.cache/composer \
  docker run --rm --interactive --tty \
  --env COMPOSER_HOME \
  --env COMPOSER_CACHE_DIR \
  --volume $COMPOSER_HOME:$COMPOSER_HOME \
  --volume $COMPOSER_CACHE_DIR:$COMPOSER_CACHE_DIR \
  --volume $PWD:/app \
  %%IMAGE%% install
```

### Filesystem permissions

By default, Composer runs as root inside the container. This can lead to permission issues on your host filesystem. You can work around this by running the container with a different user:

```console
$ docker run --rm --interactive --tty \
  --volume $PWD:/app \
  --user $(id -u):$(id -g) \
  %%IMAGE%% install
```

### Private repositories / SSH agent

When you need to access private repositories, you will either need to share your configured credentials, or mount your `ssh-agent` socket inside the running container:

```console
$ docker run --rm --interactive --tty \
  --volume $PWD:/app \
  --volume $SSH_AUTH_SOCK:/ssh-auth.sock \
  --env SSH_AUTH_SOCK=/ssh-auth.sock \
  %%IMAGE%% install
```

**Note:** This currently does not work on OSX, see [docker/for-mac#410](https://github.com/docker/for-mac/issues/410).

When combining the use of private repositories with running Composer as another user, you might run into non-existent user errors (thrown by ssh). To work around this, simply mount the host passwd and group files (read-only) into the container:

```console
$ docker run --rm --interactive --tty \
  --volume $PWD:/app \
  --volume $SSH_AUTH_SOCK:/ssh-auth.sock \
  --volume /etc/passwd:/etc/passwd:ro \
  --volume /etc/group:/etc/group:ro \
  --env SSH_AUTH_SOCK=/ssh-auth.sock \
  --user $(id -u):$(id -g) \
  %%IMAGE%% install
```

# Troubleshooting

### PHP versions

Our image is aimed at quickly running Composer without the need for having a PHP runtime installed. You should not rely on the PHP version in our container. We do not provide a Composer image for each supported PHP version because we do not want to encourage using Composer as a base image or a production image.

Suggestions:

-	use [`--ignore-platform-reqs`](https://getcomposer.org/doc/03-cli.md#install-i):

	```console
	$ composer install --ignore-platform-reqs
	```

-	specify the target [platform](https://getcomposer.org/doc/06-config.md#platform) in your `composer.json`:

	```json
	{
	  "config": {
	    "platform": {
	      "php": "7.1.3"
	    }
	  }
	}
	```

### PHP extensions

We aim to deliver an image that is as lean as possible, built for running Composer only. Sometimes dependencies or Composer [scripts](https://getcomposer.org/doc/articles/scripts.md) require the availability of certain PHP extensions.

Suggestions:

-	pass the `--ignore-platform-reqs` and / or `--no-scripts` flags to `install` or `update`:

	```console
	$ docker run --rm --interactive --tty \
	  --volume $PWD:/app \
	  %%IMAGE%% install --ignore-platform-reqs --no-scripts
	```

-	create your own buid image and [install](https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md) Composer inside it.

	**Note:** Docker 17.05 introduced [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/), simplifying this enormously:

	```dockerfile
	COPY --from=%%IMAGE%% /usr/bin/composer /usr/bin/composer
	```
