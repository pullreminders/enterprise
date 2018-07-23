## Overview

This repo contains instructions on setting up Pull Reminders on-premises.

1. [Choose a host](#choose-a-host)
- [Create a Slack App](#create-a-slack-app)
- [Create a GitHub App](#create-a-github-app)
- [Setup Pull Reminders](#setup-pull-reminders)

## Choose a host

Pull Reminders needs to run on a server within the same security context as your GHE instance so that it can send and receive requests from GitHub. For testing purposes this could just be a computer in your office, but a AWS/GC/Azure instance is ideal.

In terms of security – Pull Reminders integrates with GHE as a [GitHub App](https://developer.github.com/enterprise/2.13/apps/about-apps/#about-github-apps). This means that API permissions and repo access are granted by you when you install the app to one of your GitHub organizations. Pull Reminders requires read access to pull request metadata, repo list, and members. It does not require any read or write access to code. You'll be happy to know that your data won't leave your network other than the API requests to Slack. There's no telemetry or pingback to our public servers.

## Create a Slack App

Pull Reminders requires a Slack app in order to post messages. Follow the steps below to create a Slack app for your instance of Pull Reminders. You'll also come back later to make a few more updates after setting up your Pull Reminders instance.

1. Create a Slack app by going to https://api.slack.com/apps and clicking "Create New App"
2. Name the app "Pull Reminders" and set the "Development Slack Workspace" to your team's workspace
3. Set the app icon to https://pullreminders.com/app-icon.png
4. Next, go to the "OAuth & Permissions" tab and select the following scopes:
    
    ```
    channels:read
    chat:write:bot
    groups:read
    bot
    identity.basic
    identity.email
    identity.team
    user.profile:read
    team:read
    ```
    
5. Go to the "Bot Users" tab and create a bot user with the name "Pull Reminders" and username "pullreminders".

## Create a GitHub App

Pull Reminders requires a GitHub app in order to integrate with your GitHub Enterprise instance. Follow the steps below to create a GitHub App. You'll also come back later to make a few more updates after setting up your Pull Reminders instance.

1. Go to your GitHub "Settings" then "Developer Settings" then "GitHub Apps", then click "New GitHub App"
2. Set the app icon to https://pullreminders.com/app-icon.png
3. Under the "Permissions & webhooks" tab, set the following permissions:
    ```
    Issues:               Read-only
    Repository metadata:  Read-only
    Pull requests:        Read-only
    Organization members: Read-only
    ```
    
    You'll also want to subscribe to the following events:
    
    ```
    Issue Comment
    Issues
    Pull request
    Pull request review
    Pull request review comment
    Membership
    Organization
    ```
  
4. Finally, though not required, you'll want to go to the "Advanced" tab and transfer ownership of the GitHub App to a superuser or an organization. If it is owned by your individual user account the app can become available if your account is deleted or deactivated.


## Setup Pull Reminders

### MacOS

*In the future I'll have the app bundled in a Docker image so that these steps can be skipped.*

1. First install [Homebrew](https://brew.sh/), the "missing package manager for macOS":

    ```
    $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

2. Install Postgres with the command `$ brew install postgres` and make sure you can create a database with `$ createdb testdb`

3. Install Redis with the command `$ brew install redis` and make sure you can start it with `$ redis-server`

4. Install rbenv by following the [instructions in their README](https://github.com/rbenv/rbenv#homebrew-on-macos). Then install Ruby v2.5.0 with the command `$ rbenv install 2.5.0`. Verify that its been installed by checking your Ruby version in the command line `$ ruby version`.
