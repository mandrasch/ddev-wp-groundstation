# DDEV pull-wp

⚠️ Status: Work in progress, use at own risk 👷‍♀️⚠️

## Description

Scared a wordpress update will break your old site? 

This project contains [custom DDEV commands](https://ddev.readthedocs.io/en/stable/users/extend/custom-commands/) for pulling a live website to your local machine with just a single command (at least, this is the goal ;-)). 

In this local instance you can safely experiment with your wordpress site, e.g. perform updates and check for errors - without breaking the live site.

*See [".ddev/commands/web"](https://github.com/programmieraffe/ddev-pull-wp/tree/main/.ddev/commands/web) for implementation.*

## Demo (Screencast)

**[https://www.youtube.com/watch?v=9V9DmjIlrbI](https://www.youtube.com/watch?v=9V9DmjIlrbI)**

## Prerequisites

- [DDEV](https://www.ddev.com/ddev-local/) installed on your local machine
- [updraftplus premium](https://updraftplus.com/shop/updraftplus-premium/) license
- SSH and rsync on remote server/webspace (live website)


## Install / Setup

1. **Clone this repository, open folder in terminal**

2. **Install fresh wordpress locally**

    ```shell
    ddev install-wp
    ```

    *This command will automatically install wordpress and activate the .zip file need to activate updraftplus for addons, see: https://updraftplus.com/support/installing-updraftplus-premium-your-add-on/].* 
      
    At the end of the installation, you will be prompted for an admin password.

3. **Login into local wordpress**

    Login to https://pull-wp.ddev.site/wp-admin/ with user "admin" and your chosen password.
    
    [Use `ddev launch` to open the local site directly in browser]

4. **Login to updraftplus premium  ($)** 

    Activate updraftplus premium license with your credentials. To do that, navigate to Wordpress Dashboard > Settings > updraftplus:

    https://pull-wp.ddev.site/wp-admin/options-general.php?page=updraftplus
    
    ![Screenshot updraftplus dashboard - add credentials in Connect with updraftplus account](screenshots/screenshot_updraftplus_connect.png)
    
5. **Activate "all addons" to enable WPCLI and Migrator feature**

    Make sure to click "activate it on this site":

    ![Screenshot updraftplus dashboard - premium license activated](screenshots/screenshot_updraftplus_premium_activate.png)
    
6. **Create a local backup**

    We create a local backup (this will be useful later) after successful activation of updraftplus premium:
    
    ```shell
    ddev create-local-backup
    ```
    
    That's it, all setup. 

## Pull a remote site

1. **Install updraftplus premium on remote wordpress site**

    Install updraftpflus with this [.zip file](https://updraftplus.com/wp-content/uploads/updraftplus.zip) (Source: [Updraftplus Docs](https://updraftplus.com/support/installing-updraftplus-premium-your-add-on/)
    
    Activate updraftplus premium by logging in with your updraftplus account
    
    ![Screenshot updraftplus dashboard - add credentials in Connect with updraftplus account](screenshots/screenshot_updraftplus_connect.png)
    
    and click "activate it on this site" as well:
    
    ![Screenshot updraftplus dashboard - premium license activated](screenshots/screenshot_updraftplus_premium_activate.png)

2. **Pull remote site backup and restore (migrate) it**

    Now we can pull the backup from the live website to our local site:
    
    ```shell
    ddev pull-wp sshusername@host.xyz /html/wordpress
    ```
    
    *This command connects to the remote live website via SSH, creates a backup via updraftplus CLI, rsyncs the backup files to local DDEV, restores it with help of updraftplus Migrator and CLI feature.*
    
    Example command for [Uberspace webspace](https://uberspace.de/en/):
    
    ```shell
    ddev pull-wp UBERSPACE_USER@draco.uberspace.de /var/www/virtual/UBERSPACE_USER/html/
    ```
    
    You can find out the wordpress path of your live site via
    
    1. Login to your site via ssh
    2. Change directory to wordpress folder
    3. Check if WPCLI is available with "wp core version"
    4. Get the wordpress folder path with "pwd" command (print working directory)

4. **Open and test updates locally**

    Open `https://pull-wp.ddev.site` to see the migrated site which now runs on your local machine.
    
    The local login credentials are replaced by the credentials of the live website. You can now test a bigger wordpress update in peace - without breaking the live website.
     
    A good practice could be to use tools like Disable Emails, WP Debug Bar, etc. Activate debug log and install plugin Debug bar e.g.:
    
    ```shell
    ddev exec wp config set WP_DEBUG true --raw --path=wordpress
    ddev exec wp config set WP_DEBUG_LOG true --raw --path=wordpress
    ddev exec wp config set WP_DEBUG_DISPLAY false --raw --path=wordpress
    ddev exec wp plugin install debug-bar --activate --path=wordpress
    ```
    
    Or use the [WP Debugging](https://wordpress.org/plugins/wp-debugging/) plugin.

5. **Run updates on live website**

    If the update works locally, you can just run updates on your live site as well. 

## Reset

1. **Reset your local site**

    If you have created a local restore point in the beginning (ddev create-local-backup), you can restore it here:

    ```shell
    ddev reset-wp
    ```
    If you haven't, things maybe a little bit more complicated:
    
    After a restore process you need to activate the updraftplus premim license or CLI-addon/Migrator-addon again in WP dashboard unfortunately. Otherwise you'll get "Error: 'updraftplus' is not a registered wp command". 

2. **(optional) Delete transferred backups**

    If you want to cleanup the transferred backups, run:

    ```shell
    ddev delete-backups
    ```
    
    (This will not delete your local restore point from installation)
    
3. **Start with ddev pull-wp again**

## Advanced

### Local PHP/MySQL version

See .ddev/config.yaml to change the local docker enviroment to be similiar to your remote webspace/server. You can also use nginx instead of apache.

### Working with git?

If you have a theme checked out with git on remote website, this will be transferred to the local enviroment. Updraftplus does not delete .git-folders while migration (which is nice).

### Forgot password?

Run following command:

```shell
ddev exec wp user update admin --prompt=user_pass --path=wordpress/
```

### Symlink plugin/themes in fresh wp

    Beware: Experimental

    1. Clone a theme or plugin via git into symlinked-themes/ or symlinked-plugins/

    2. Setup symlink

    This will delete wp-content/themes/mychild-theme and replace it with symlink:

    ```shell
    ddev setup-symlink theme mychild-theme
    ```

    See ".ddev/commands/web/setup-symlink" for details.

### Full/hard reset
  
Drop database datables and remove wordpress/-directory:

```shell
ddev delete-wp
```
If you need to remove other non-tracked files:
(Caution: also deletes ignored files + symlinked-/ folders)

```shell
git clean -fdx
```
Delete complete DDEV containers:

```shell
ddev delete -O
```

If you made changes to DDEV files from this git repo, which you want to delete: 

```
git reset --hard`
```