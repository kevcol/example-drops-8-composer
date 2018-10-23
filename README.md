# Pantheon Thunder 8 Composer without CI

This repository is a start state for a Composer-based Drupal 8 Thunder site to be installed on [Pantheon](https://pantheon.io/) without CI. It is based on https://github.com/pantheon-systems/example-drops-8-composer

**Caveat:** There are a fair amount of clunky, manual steps, and I'm new at this, so suggestions and pull requests are welcome!


## Installation

### Prerequisites

* [A Pantheon account](https://dashboard.pantheon.io/register)
* [Terminus, the Pantheon command line tool](https://pantheon.io/docs/terminus/install/)
* [The Terminus Build Tools Plugin](https://github.com/pantheon-systems/terminus-build-tools-plugin)
* Not strictly required, but it helped me to have a second version of Thunder running locally which was installed by the [official Thunder composer.json approach](https://github.com/BurdaMagazinOrg/thunder-project/blob/2.x/README.md)


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


### Cloning pantheon-thunder-8-composer Locally

Instead of setting up `composer.json` manually, it is easier to start with the `pantheon-thunder-8-composer` repository.

1. Clone the pantheon-thunder-8-composer repository locally:

```
git clone git@github.com:kevcol/pantheon-thunder-8-composer.git $PANTHEON_SITE_NAME
```

2. `cd` into the cloned directory:

```
cd $PANTHEON_SITE_NAME
```

### Updating the Git Remote URL
Store the Git URL for the Pantheon site created earlier in a variable:

```
export PANTHEON_SITE_GIT_URL="$(terminus connection:info $PANTHEON_SITE_NAME.dev --field=git_url)"
```

Update the Git remote to use the Pantheon site Git URL returned rather than the `pantheon-thunder-8-composer` GitHub URL:

```
git remote set-url origin $PANTHEON_SITE_GIT_URL
```

### Managing Drupal with Composer

#### Downloading Drupal Dependencies with Composer
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

3. Delete git submodules

When possible, use tagged versions of Composer packages. Untagged versions will include `.git` directories, and the [Pantheon platform is not compatible with git submodules](https://pantheon.io/docs/git-faq/#does-pantheon-support-git-submodules), so you may need to find and delete them.

Run the following in your project directory...

```
find . -name ".git"
```
... and manually delete any `git` and `.gitignore` files in modules (eg: fb_instant_articles, views_load_more) or under the `/vendor` directory.  The only git repo should be the main one for your project.

(**Note:** You can add those `git` submodules back after you push your commit up to Pantheon. To do this, remove the `vendor` directory and run `composer install`.)

5. Set the site to git mode:

```
terminus connection:set $PANTHEON_SITE_NAME.dev git
```

6. Add and commit the code files. A Git force push is necessary because we are writing over the empty repository on Pantheon with our new history that was started on the local machine. Subsequent pushes after this initial one should not use `--force`:

```
git add .
git commit -m 'Drupal 8 and dependencies'
git push --force
```

### Installing Drupal
Now that the code for Drupal core exists on our Pantheon site, we need to actually install Drupal.

Rather than use Terminus, go to the Pantheon UI, and click through to your site. On the **Dev** tab, set the site to `SFTP` mode click the button that says

> Visit Development Site

If you're not redirected to the install page, add `/install.php` after the dev URL. With luck, you should see the Thunder installer admin.  

#### Manual fiddling
Back up in the prerequisites, I suggested you have a copy of Thunder running locally installed via the [official Thunder composer.json approach](https://github.com/BurdaMagazinOrg/thunder-project/blob/2.x/README.md). This came in handy when I hit a validation error in the Thunder installation UI.

For reasons I haven't yet sussed out, I hit a validation error during in the UI because I was missing `scheduler_content_moderation_integration`.

So I manually copied the `scheduler_content_moderation_integration` directory from my local Thunder installation into my Pantheon codebase, committed the change and pushed it up.

Once that was included, I was able to complete the installation via the UI.

But the Pantheon-hosted site had some problems because the libraries directory didn't exist.  I saw these errors on the Drupal status report:

```
Dropzonejs requires the dropzone.min.js library. Download it (https://github.com/enyo/dropzone) and place it in the libraries folder (/libraries)

Make sure that the select2 lib is placed in the library folder. You can download the release of your choice from GitHub.

Please download at least v1.4.6 of the shariff library and place it in one of your libraries folders. So that for example the js file is available under DRUPAL_ROOT/libraries/shariff/shariff.min.js.

The Slick library should be installed at /libraries/slick/slick/slick.min.js, or any path supported by libraries.module if installed.
```

I went back to my local version of Thunder and copied the entire `/libraries` directory to my Pantheon codebase (under `/web`), committed the code and pushed it up.

Voila!: Thunder 8 on Pantheon (via the scenic route)
