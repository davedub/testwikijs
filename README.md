### How to Get a [Wiki.js](https://wiki.js.org/) app running on Heroku & Github

#### Overview

Setting up a Wiki JS app on Heroku, with individual pages within the app pulled from Github, is [documented by Requarks](https://docs.requarks.io/wiki/), which created Wiki JS, but I found it confusing to follow. This summary attempts to remove some of the mystery of what finally worked for me.

#### Setup

1. 	If you don't have a Heroku account, create one. Apps that run on shared "dynos" (which is what the no-charge tier plan offers) are free. 

2. 	If you don't have git, install it. After that, create a Github account. GitHub is free to use for public and open source projects.  

3.	Create an empty Github repository (do NOT check the box "Initialize this repository with a README"). This is where you will store your wiki's draft pages (i.e., markdown files). If you want to keep them hidden while working on your wiki (which can be set to private mode later, so no one will see it until you flip the switch to take it live), you need a paid Github account (and then, make this a private repository).

4.	Set up and initialize an empty git repository on your local machine, and add this Github repo as its remote origin

```
$ mkdir testwikijs
$ git init
$ git remote add origin https://github.com/davedub/testwikijs.git
```
make a README file on your local machine and push it to your Github repo.

```
$ touch README.md
// add text in the nano text editor, the save in file
$ nano README.md 
$ git add .
$ git commit -m 'add readme'
$ git push origin master
```

5. 	Now, go to the Github app for wikiJS (https://github.com/Requarks/wiki) and use the "Deploy to Heroku" button to create a new Wiki.js app on Heroku. This will launch Heroku.

6.	Now that you are on a Heroku deploy page, you will walk through a few steps to create the Wiki.JS app. Name it the same as your Github repo - I am using "testwikijs" - and make note of the fact that an mLab MongoDB add-on is provisioned (Sandbox version - Free) and that certain configuration variables ("config vars") are set. You can modify these later, but you may want to change the WIKI_TITLE to something other than "Wiki" (I changed mine to "Wiki.js Test using Heroku-Github OAuth pipeline").  Also note that in the process of creating the app, and configuring its environment, you will see an autoscrolled output box. This is just taking the deploy process built into the "Deploy to Heroku" button step by step. For now, click the button that says "Manage App" to view that dashboard.

7. Brief detour here ... on the dashboard, click the "More" tab (in the upper right, next to the "Open app" tab) and choose "View logs". You will see info like this

```
app[web.1]: [32minfo[39m: [AGENT] Checking Git repository...
app[web.1]: [31merror[39m: [AGENT] Unable to fetch from git origin!
app[web.1]: [31merror[39m: [AGENT] Unable to push changes to remote Git repository!
app[web.1]: [31merror[39m: [AGENT] One or more jobs have failed:  
app[web.1]: { code: [33m1[39m,
app[web.1]:   message:
app[web.1]:    [32m'`git --git-dir=repo/.git --work-tree=repo pull origin master` failed with code 1'[39m,
app[web.1]:   stderr:
app[web.1]:    [32m"remote: Invalid username or password.\nfatal: Authentication failed for 'https://marty:MartyMcFly88@github.com/Organization/Repo/'\n"[39m,
app[web.1]:   stdout: [32m''[39m }
```
 
This indicates that your Wiki.js app, while deployed, cannot yet fetch articles (markdown files) from the Github repository that you will set up to manage (and persistently) your wiki. Because dynos on Heroku's free tier are shared and spin up only when a GET request is made to the URL for your Heroku app), any changes you make to your wiki ***from inside your Heroku app*** will disappear the next time you launch it. For the content to persist, it must be pulled from your Github repo. 

8. Go to the developer settings page of your Github account (https://github.com/settings/developers) and create a new OAuth app. This is what will give you a connection between the markdown files you push to your Github repo and your WikiJS app on Heroku. Give the OAuth app an "Application name" that is the same as your repo (testwikijs). Set the "Homepage URL" to that of your Heroku app (https://testwikijs.herokuapp.com). Make sure to specify "Authorization callback URL" as `https://testwikijs.herokuapp.com/login/github/callback`. Now, click on the "Register application" (green) button. This will return you to the OAuth app page for your newly registered application, which includes an auto-generated "Client ID" (20 alphanumeric (lower case) characters) and "Client Secret" (20 alphanumeric (lower case) digits). 

#### [RANDOMIZED](https://www.random.org/strings/) EXAMPLE:
```
Client ID
vevsxhj61eem9pyjy40o
Client Secret
9gj1lj71u7205w1wl4xchhow3b1brrezbpnbh4bf
```
SIDE NOTE here: The [Requarks documentation](https://docs.requarks.io/wiki/install/installation/authentication) suggests that you copy both Client ID and Client Secret as will need to past them into your "config.yaml." 

> Under the auth section of your config.yml, you can now enter the required info:
>
```
github:
    enabled: true
    clientId: GITHUB_CLIENT_ID
    clientSecret: GITHUB_CLIENT_SECRET
```
This might make sense if you fetch a local copy of the project comprising your deployed app off Heroku, in which case you could modify the config.yaml file but I don't see a need to do that here. 

9.	Go to the "Settings" tab in your Heroku app's dashboard and click the "Reveal Config Vars" button. Notice that it is an array of key-value pairs with default values (except whichever one you may have changed during the deploy step). Here, you need to make several changes:

* WIKI_GIT_URL becomes the URL of your Github repository (https://github.com/davedub/testwikijs). 
* WIKI_HOST becomes the URL of your Heroku app (https://testwikijs.herokuapp.com)
* WIKI_GIT_PASSWORD becomes your Github account password
* WIKI_GIT_USERNAME becomes your Github account username

Also change WIKI_IS_PUBLIC to true just to make sure everything is working as expected. You can return it to FALSE later as you start writing real content for your wiki.

Note as well on this page that the Settings page has an "Info" tab that shows the "Slug size" of your app. This is one of [various limits imposed](https://devcenter.heroku.com/articles/limits) on apps deployed on Heroku. If your app's requirements exceed these limits, consider switching to a more robust platform.

10.	Create home.md in your local machine, then push it to your Github branch. Wait a few minutes before trying to entering your Heroku app's URL in the URL of your browser, to see whether your content pipeline works. Log in to your Heroku app (the email address is specified in config vars and the default password is "admin123", which you can change later). 

Hopefully, it works for you (as it did for me) just fine. 