# Pantheon Thunder 8 Composer without CI

This repository is a start state for a Composer-based Drupal 8 Thunder site to be installed on [Pantheon](https://pantheon.io/) without CI. It is based on https://github.com/pantheon-systems/example-drops-8-composer

**Caveat:** There are a fair amount of clunky, manual steps, and I'm new at this, so suggestions and pull requests are welcome!


## Installation

### Prerequisites

Before running the `terminus build:project:create` command, make sure you have all of the prerequisites:

* [A Pantheon account](https://dashboard.pantheon.io/register)
* [Terminus, the Pantheon command line tool](https://pantheon.io/docs/terminus/install/)
* [The Terminus Build Tools Plugin](https://github.com/pantheon-systems/terminus-build-tools-plugin)


### Creating the Pantheon Site:
Modified from https://pantheon.io/docs/guides/drupal-8-composer-no-ci/

To begin, we’ll want to start a brand new Drupal 8 site on Pantheon from our empty upstream. This upstream is different from the Drupal 8 upstream in that it does not come with any Drupal files. As such, you must use Composer to download Drupal.

Before we begin choose a machine-friendly site name. It should be all lower case with dashes instead of spaces. I'll use d8-thunder-no-ci but choose your own. Once you have a site name export it to a variable for re-use.

```
export PANTHEON_SITE_NAME="d8-thunder-no-ci"
```

You should also be authenticated with Terminus. See the [Authenticate into Terminus](https://pantheon.io/docs/machine-tokens/#authenticate-into-terminus) section of the [machine tokens](https://pantheon.io/docs/machine-tokens/) documentation for details.

Create a new Pantheon site with an empty upstream.

```
terminus site:create $PANTHEON_SITE_NAME 'My D8 Composer Site' empty
```

**Note** you can also add the `--org` argument to `terminus site:create` if you would like the site to be part of an organization. See `terminus site:create -h` for details and help.


###Cloning pantheon-thunder-8-composer Locally

Instead of setting up `composer.json` manually, it is easier to start with the `pantheon-thunder-8-composer` repository.

1. Clone the pantheon-thunder-8-composer repository locally:

```
git clone git@github.com:kevcol/pantheon-thunder-8-composer.git $PANTHEON_SITE_NAME
```

2. `cd` into the cloned directory:

```
cd $PANTHEON_SITE_NAME
```

###Updating the Git Remote URL
Store the Git URL for the Pantheon site created earlier in a variable:

```
export PANTHEON_SITE_GIT_URL="$(terminus connection:info $PANTHEON_SITE_NAME.dev --field=git_url)"
```

Update the Git remote to use the Pantheon site Git URL returned rather than the `pantheon-thunder-8-composer` GitHub URL:

```
git remote set-url origin $PANTHEON_SITE_GIT_URL
```

###Managing Drupal with Composer

**Note:** When possible, use tagged versions of Composer packages. Untagged versions will include `.git` directories, and the [Pantheon platform is not compatible with git submodules](https://pantheon.io/docs/git-faq/#does-pantheon-support-git-submodules). These instructions include a step where you'll manually remove some `.git` directories, be sure to put them back again after you push your commit up to Pantheon (see instructions below). To do this, remove the vendor directory and run composer install.

####Downloading Drupal Dependencies with Composer
Normally the next step would go through the standard Drupal installation. But since we’re using Composer, none of the core files exist yet. Let’s use Composer to download Drupal core.

1. Update Composer to download the defined dependencies:

```
composer update
```

2. Let's take a look at the changes:

```
git status
```



This may take a while as all of Drupal core and its dependencies will be downloaded. Subsequent updates should take less time.

------
