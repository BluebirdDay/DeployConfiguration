# Hipex Deploy Configuration
These documentation is the guide to painlessly setup an automated deploy on the Hipex B.V. platform.
The repository contains:

- Configuration objects
- Documentation

The deploy image is based on https://deployer.org/ and is 100% compatible with its recipe's and functions. Hipex uses
configuration objects to provide some form of autocompleteion. These configuration objects can be found in this
repository. The hipex wrapper will use the configuration set to create a set of deploy recipe's and hosts that
can be overwritten / extended by the projects `deploy.php`.

## Whats inside?
- Deployer configuration hosts / tasks
- Hipex server setup
- Email / New Relic notification
- CloudFlare flush

## Configuration
1. Composer `require hipex/deploy-configuration --dev` package. Only needed when you want to have autocomplete in your `deploy.php`
file.
2. Copy the deploy templates inside the root of your project as `deploy.php`. You can find the template in
[templates/deploy.php](./templates/deploy.php).
3. Setup your CI server
    1.  GitLab CI [templates/.gitlab-ci.yml](./templates/.gitlab-ci.yml).
    3.  Bitbucket [templates/bitbucket-pipelines.yml](./templates/bitbucket-pipelines.yml).
4. Your first build will fail due to missing configuration. Login to the server and depending on your project file edit
the `app/etc/env.php` or `app/etc/local.xml`. You will find these files in `~/domains/<domain>/application/shared/`.  

## Build steps
1. `build:update` Checkout the code and run composer installer
2. `build:compile` Run frontend build and compilation tasks and other `Configuration::getBuildCommands` commands
3. `build:package` Package code into `tgz` archive
4. `deploy:prepare_release` Prepare deployment folders on the server(s)
5. `deploy:copy` Copy build files to server(s)
6. `deploy:deploy` Run database migrations and other `Configuration::getDeployCommands` commands
7. `deploy:link` Set symlinks to new version
8. `deploy:after` Flush caches and send notification and other `Configuration::getAfterDeployCommands` commands

All these steps are regular deployer tasks so the can be extended or even overwritten per project.

## Required environment variables
Some specific environment variables are required to allow the deploy image access to the git repository
or to be able to send out notifications.

### Required
- `SSH_PRIVATE_KEY` Unencrypted SSH key. The key needs to have access to: main git repository, private packages
and the SSH user.

### Composer authentication
These variables are only required if Magento composer repository authentication is not configured using `auth.json`. 
- `COMPOSER_AUTH` base64 encoded content of `auth.json`. Can be fetched using `cat auth.json | base64`.

### Optional
Email notifications
- `SMTP_SERVER`
- `SMTP_USER` 
- `SMTP_PASS`
- `NOTIFICATION_EMAIL_TO` Comma separated list of email address to send notification to
- `NOTIFICATION_EMAIL_FROM`

NewRelic deploy mark (https://deployer.org/recipes/newrelic)
- `NEWRELIC_APP_ID`
- `NEWRELIC_API_KEY`

Slack (https://deployer.org/recipes/slack)
- `SLACK_WEBHOOK`
