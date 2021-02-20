# DDEV pull-wp

⚠️ Status: Work in progress, use at own risk 👷‍♀️⚠️

Scared a wordpress update will break your old site? Want to play around with your wordpress site locally? DDEV pull-wp contains DDEV commands for pulling a live website to your local machine with just a single command (at least, this is the goal ;-)). 


## Prerequisites

- [DDEV](https://www.ddev.com/ddev-local/) installed on your local machine
- [updraftplus premium](https://updraftplus.com/shop/updraftplus-premium/) license
- SSH and rsync on remote server/webspace (live website)

## Install / Setup

Screencast:



1. **Clone this repository, open folder in terminal**

2. **Install fresh wordpress locally (installs updraft automatically)**

    ```shell
    ddev install-wp
    ```

    This command will automatically install wordpress and activate the .zip file need to activate updraftplus for addons, see: https://updraftplus.com/support/installing-updraftplus-premium-your-add-on/]. 
      
    At the end of the installation, you will be prompted for an admin password.

3. **Login into local wordpress**

    Login to https://pull-wp.ddev.site/wp-admin/ with user "admin" and your chosen password.
    
    [Use `ddev launch` to open the local site directly in browser]

4. **Activate updraftplus premium  ($)** 

    Activate updraftplus premium license or paid license for updraftplus CLI and Migrator in the dashboard:

    https://pull-wp.ddev.site/wp-admin/options-general.php?page=updraftplus
    
    ![Screenshot updraftplus dashboard - add credentials in Connect with updraftplus account](screenshot_updraftplus_connect.png)

    ![Screenshot updraftplus dashboard - premium license activated](screenshot_updraftplus_premium_activated.png)
    
    (See https://updraftplus.com/support/installing-updraftplus-premium-your-add-on/ for more details)
    
5. **Create a backup point locally to enable easier resets**

    If the updraftplus license is activated locally, we create a restore point (this will be useful later):

    ```shell
    ddev create-local-backup
    ```
    
    That's it, all setup. 

## Pull a remote site

1. **Install updraftplus (free/premium) on remote wordpress site**

    On the remote site it's best to use the updraftplus .zip file as well from here: https://updraftplus.com/support/installing-updraftplus-premium-your-add-on/ - but you don't have to activate/connect anything. Free version is enough. 
    
    If you have enough licenses, it's more comfortable to have updraftplus premium activated on all sites (remote / local).

2. **Create backup on remote site**

    *If you have premium/WPCLI-addon installed on your remote/live site, you can skip this step.*

    ![Screenshot updraftplus dashboard - Backup screen](screenshot_updraftplus_backup_now.png)
  
    Just create a backup with "Backup now" in the Wordpress Dashboard of your live website.

3. **Pull remote site backup and restore (migrate) it**

    Now we can pull the backup from the live website to our local site. The ssh username and host is needed as well as the path where wordpress is installed on the remote webspace. This command will fetch the latest backup from remote to local.
      
    If your remote site has updraftplus WPCLI activated, this script also can start a fresh backup on your remote site:
    
    ```shell
    ddev pull-wp sshusername@host.xyz /html/wordpress
    ```
    
    (This command connects to the remote live website via SSH, rsync's the backup files to local DDEV, restores it with help of updraftplus Migrator and CLI. You need to provide SSH information and path to wordpress installation on remote server.)

4. **Open and test updates locally**

    Open `https://pull-wp.ddev.site` to see the migrated site which now runs on your local machine.
    
    The local login credentials are replaced by the credentials of the live website. You can now test a bigger wordpress update in peace, without breaking the live website.
     
    A good practice could be to use tools like Disable Emails, WP Debug Bar, etc. Activate debug log and install plugin Debug bar e.g.:
    
    ```shell
    ddev exec wp config set WP_DEBUG true --raw --path=wordpress
    ddev exec wp config set WP_DEBUG_LOG true --raw --path=wordpress
    ddev exec wp config set WP_DEBUG_DISPLAY false --raw --path=wordpress
    ddev exec wp plugin install debug-bar --activate --path=wordpress
    ```

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

## Usage scenarios

1. updraftplus CLI local <-> free updraftplus on remote site (🤖 <-> 📁)

    In this mode you create the backup on your remote site with the free updraftplus using the WP dashboard. Afterwards you can sync it to your local machine via the local WPCLI addon and Migrator addon (both $) of updraftplus activated on your local machine.

2. updraftplus CLI local <-> updraftplus CLI remote (🤖 <-> 🤖)
    
    You'll need activated premium (or addon licenses) on local and remote machine(s) for updraftplus. 
    
    In CLI<->CLI (commandline-to-commandline) mode you can also create backups on remote site from your local machine, no need to visit the WP dashboard of your remote site. You can also get the <nonce>-identifier for the latest backup, which saves time syncing. 
  
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