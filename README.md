## Overview

This repo contains instructions on setting up Pull Reminders on-premises for integration with GitHub Enterprise.

1. [Set up your infrastructure](#set-up-your-infrastructure)
2. [Create a Slack App](#create-a-slack-app)
3. [Create a GitHub App](#create-a-github-app)
4. [Set up your Pull Reminders instance](#set-up-your-pull-reminders-instance)
5. [Administration](#administration)

## Set up your infrastructure

Pull Reminders requires a Postgres database and a host for running a Docker image that contains the Pull Reminders application server. For optimal performance, your host should have at least 2 GB RAM and your Postgres database (run separately, e.g., AWS RDS or Google Cloud SQL) should have at least 2 GB RAM and 10 GB storage. Your Postgres version should be 10 or higher. For handling HTTPS connections we recommend using an HTTP proxy (e.g., nginx) or load balancer with SSL configured.

Pull Reminders needs to run within the same security context as your GitHub Enterprise (GHE) instance so that it can send and receive requests from GHE. Pull Reminders also needs to be able to make external requests to Slack's API, but otherwise Pull Reminders does not send data or make requests outside of your network. 

## Create a Slack App

Pull Reminders requires a Slack app in order to post messages. Follow the steps below to create a Slack app for your instance of Pull Reminders.

1. Create a Slack app by going to https://api.slack.com/apps and clicking "Create New App"
2. Name the app "Pull Reminders" and set the "Development Slack Workspace" to your team's workspace
3. Set the app icon to the [image provided in this repo](./icon.png)
4. Next, go to the "OAuth & Permissions" tab and select the following scopes:
    
    ```
    channels:read
    chat:write:bot
    groups:read
    bot
    identity.basic
    identity.email
    identity.team
    users.profile:read
    team:read
    ```

5. Under the same "OAuth & Permissions" tab, add a new Redirect URI. It should be your hostname plus the path `/auth/slack/callback`, e.g. `https://pullreminders.myco.com/auth/slack/callback`. If you don't know your hostname yet, remember to come back and set this later.
6. Go to the "Bot Users" tab and create a bot user with the name "Pull Reminders" and username "pullreminders".
7. After setting up your Slack app, **do not** install the app to your Slack workspace from Slack's website or submit it to the Slack App Directory. You will install the Slack App through Pull Reminders once your instance is set up.
8. Slack does not currently offer static IPs for whitelisting, but you can [whitelist your IPs in Slack](https://api.slack.com/docs/oauth-safety#ip_whitelisting) if you would like.

## Create a GitHub App

Pull Reminders requires a [GitHub App](https://developer.github.com/enterprise/2.13/apps/about-apps/#about-github-apps) in order to integrate with your GitHub Enterprise instance. Follow the steps below to create a GitHub App. You'll also come back later to make a few more updates after setting up your Pull Reminders instance.

1. Go to your GitHub "Settings" then "Developer Settings" then "GitHub Apps", then click "New GitHub App"
2. Set the fields, permissions, and event subscriptions outlined below. You can always change these later after saving. Note that several of these settings include your Pull Reminders instance hostname, so if you don't know it yet be sure to come back later and update your settings.
    
    Fields:
    
    ```
    GitHub App name:      Pull Reminders
    User auth callback:   http://pullreminders.myhost.com/auth/github/callback
    Setup URL:            http://pullreminders.myhost.com/installs
    Webhook URL:          http://pullreminders.myhost.com/webhooks/github
    ```
    
    Permissions:

    ```
    Issues:               Read-only
    Repository metadata:  Read-only
    Pull requests:        Read-only
    Organization members: Read-only
    ```
    
    Subscribe to events (note – the `Member`option is not available for earlier versions of GitHub Enterprise):
    
    ```
    Issue Comment
    Issues
    Pull request
    Pull request review
    Pull request review comment
    Member
    Membership
    Organization
    ```
    
3. After saving your GitHub App, set your app icon to the [image provided in this repo](./icon.png) and generate a private key. You should also see OAuth credentials and your app ID. You'll use this information along with your private key in the next section.
4. **IMPORTANT:** Go to the "Advanced" tab and click the "Make public" button at the bottom. This allows Pull Reminders to be used and installed by all users of your GitHub Enterprise instance.
5. Though not required, it is recommended that you transfer ownership of the GitHub App to a superuser or organization so that the app isn't deleted if your individual GitHub account is deactivated. To do this, go under the "Advanced" tab and click the "Transfer ownership" button at the bottom.

## Set up your Pull Reminders instance

Pull Reminders is packaged as a Docker image, made available through our private portal. Follow the steps below to set up and launch your Pull Reminders instance.

1. Make sure your server host has Docker installed
2. Download the Pull Reminders docker image
3. Import it using `docker load -i pullreminders.latest.tar`
4. Create a working directory on your host `mkdir ~/pullreminders`
5. Inside this directory, create a `dockerenv` file with the variables set below. Check out a full [example dockerenv file](./dockerenv.example). `DATABASE_URL` is your Postgres connection url.

    ```
    DATABASE_URL=
    APP_HOST=
    SLACK_CLIENT_ID=
    SLACK_CLIENT_SECRET=
    GITHUB_URL=
    GITHUB_API_ENDPOINT=
    GITHUB_ENTERPRISE_VERSION=
    GITHUB_APP_URL=
    GITHUB_APP_ID=
    GITHUB_CLIENT_ID=
    GITHUB_CLIENT_SECRET=
    GITHUB_PRIVATE_KEY=
    ENABLE_ONBOARDING=1
    ```
    
    Due to limitations in Docker's [parsing of multi-line environment variables](https://github.com/moby/moby/issues/12997), you need to replace all newlines in GITHUB_PRIVATE_KEY with `\n` so it is a single line like this:

    ```
    GITHUB_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\nMIIEogIBA...
    ```

6. Start the container with your `dockerenv` file:

    ```
    docker run --rm -p 80:3000 --env-file ./dockerenv -d pullreminders
    ```

7. Browse to your instance's URL and you should be able to login. Follow the instructions presented in the app to install the GitHub App to an organization and then connect it to Slack.

8. For handling HTTPS (443) connections we recommend using an HTTP proxy (e.g., nginx) or load balancer with SSL configured.

## Administration

This section contains information on administering and maintaining your Pull Reminders instance.

#### Accessing logs

```
# Server logs
docker exec <cid> tail log/production.log -f
  
# Worker logs
docker exec <cid> tail log/sidekiq.log -f
```

#### Installing updates

We periodically release updates, provided as a new Docker image. Update your Pull Reminders instance by downloading and running the new Docker image.
