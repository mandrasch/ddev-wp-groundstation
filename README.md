# ddev-pull-wp

⚠️ Status: Preview / ddev pull-wp not implemented yet completely! ⚠️

## Description

Scared wordpress updates will break your wordpress site? Not anymore. Use the open source tool DDEV with updraftplus addons and test updates beforehand in a local dev enviroment without breaking anything on the live website. This repository contains a preconfigured DDEV with custom commands.

## Prerequisites

- DDEV installed on local machine
- SSH and rsync on remote server/webspace (not all webspaces support this)

## Install

1. **Clone this repository**

2. **Open terminal**

    Change the current directory to the cloned repository:

    ```shell
    cd ddev-pull-wp
    ```

3. **Install fresh wordpress locally**

    ```shell
    ddev install-wp
    ```

    If you need a specific language version, use: `ddev install-wp de_DE`. This will be prompted as well.

    This command will automatically install wordpress and activate the .zip file need to activate updraftplus for addons (See: https://updraftplus.com/support/installing-updraftplus-premium-your-add-on/). 

    At the end of the installation, you can set a your admin password.

4. **Login into local wordpress**

    Login to https://pull-wp.ddev.site/wp-admin/ with user "admin". You can use `ddev launch` to open your ddev site in your browser.

4. **Activiate updraftplus CLI and Migrator add-on ($)** 

    Activate paid license for updraftplus CLI and Migrator / premium in updratfplus dashboard with updraftplus account:

    https://pull-wp.ddev.site/wp-admin/options-general.php?page=updraftplus

    ![Screenshot updraftplus dashboard - add credentials in Connect with updraftplus account](screenshot_updraftplus_connect.png)

    ![Screenshot updraftplus dashboard - CLI and Migrator addon successful activated](screenshot_updraftplus_activated.png)

    (See: https://updraftplus.com/support/installing-updraftplus-premium-your-add-on/)

## Pull a remote site

1. **Install updraftplus free on remote wordpress site**

    On the remote site the .zip is not needed, just install it the standard way from https://wordpress.org/plugins/updraftplus/. 

2. **Create backup for remote site**

    Just create a backup with "backup now" and standard settings.

3. **Pull remote site backup and migrate it with a single command**

    Beware: This will delete all current changes made to wordpress in your local instance.

    ```shell
    ddev pull-wp sshusername@host.xyz /html/wordpress
    ```

    Pulls remote site backup via SSH and rsync to local DDEV, restore it with help of updraftplus Migrator and CLI. You need to provide ssh information and path to wordpress installation on remote server.

4. **Open and test updates locally**

    https://pull-wp.ddev.site
 
    A good practice could be to use tools like Disable Emails, WP Debug Bar and enabling  your WP_DEBUG and WP_DEBUG_LOG to check if updates break something.

5. **Run updates on live website**

## Advanced

### Local PHP/MySQL version

See .ddev/config.yaml to change the local docker enviroment to be similiar to your remote webspace/server. You can also use nginx instead of apache.

### Working with git?

If you have a theme checked out with git on remote website, this will be transferred to the local enviroment. Updratfplus does not delete .git folders while migration.

### Forgot autogenerated password?

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