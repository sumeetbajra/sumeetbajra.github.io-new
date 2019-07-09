---
layout: post
categories: blog
title: "Migrating from Medium to Jekyll and Netlify\_CMS"
tags:
  - jekyll
  - netlify
  - cms
  - migrate
  - blog
date: 2019-07-04T11:08:17.590Z
thumbnail: /img/final.png
---
![](/img/final.png)

We at [Botsplash](https://botsplash.com) recently migrated our blogging platform from Medium to self hosted [Jekyll](https://jekyllrb.com) site with [Netlify CMS](https://www.netlifycms.org/) to manage the content. 

Let's look at how you can migrate your blog from Medium to Jekyll and use Netlify CMS to manage the content. We will be hosting our site using Github pages in this tutorial but you can also use your custom domain which we will be discussing at the end. We shall be covering the following steps in this blog:

1. Export contents from Medium as zip
2. Create new Jekyll project
3. Import Medium articles to Jekyll
4. Integrate Netlify CMS (along with Github OAuth)
5. Using your custom domain (optional)

## 1. Export contents from Medium as zip

Medium allows you to export all your contents. You need to login to your Medium account and head to the [Settings](https://medium.com/me/settings) page. If you scroll down, you will see "Download your information" section just above "Membership". Just click the "Download .zip" button and then in the next screen, click 'Export'.

![](/img/screen-shot-2019-07-02-at-2.33.48-pm.png)

Now you should receive all your contents in zip format in your email. Go ahead and download it. If you unzip it, you can see it has all the content with comments, claps, bookmarks, drafts etc. We only need the "posts" folder for now.

## 2. Create new Jekyll project

Now we have our Medium articles exported, we need to get our website ready where we can import it and display it. For this tutorial, we are going to use an existing Jekyll [theme](https://github.com/sumeetbajra/beautiful-jekyll). Go ahead and download the project as zip in your local system.

If you instead clone the project, you will need to remove the .git folder since we will be uploading the project to our own repository later.

```
> git clone https://github.com/sumeetbajra/beautiful-jekyll
> cd beautiful-jekyll
> rm -rf .git
```

[Jekyll](https://jekyllrb.com/) is a simple, static and extendable site generator. It is based on Ruby and hence you'll need to have full Ruby development environment setup. More information can be found [here](https://jekyllrb.com/docs/installation/). 

After you have the environment setup, go into the project folder using your terminal and enter `bundle install` command. You might be prompted to enter your admin password. After the required packages are installed, go ahead and run the Jekyll project using the command `bundle exec jekyll serve`

```
> cd beautiful-jekyll
> bundle install
> bundle exec jekyll serve
```

You should see your blog running at http://localhost:4000

![](/img/screen-shot-2019-07-03-at-3.42.27-pm.png)

Copy the 'posts' folder from the 'medium-export' folder which we unzipped in the previous step to our Jekyll project folder. Note that 'posts' and '_posts' folders are different folders. 'posts' is the one containing our Medium exported blogs and '_posts' is the one used by Jekyll internally.

![](/img/screen-shot-2019-07-03-at-4.23.54-pm.png)

Note: Medium considers comments/responses as posts so the posts folder will contain the comments/responses made in the original Medium article as separate html files. Since we don't want to display them in our blog as separate entry, we can go ahead and delete them.

## 3. Import Medium articles to Jekyll

Jekyll reads posts from '_posts' folder which currently contains sample posts. We can go ahead and delete all the files inside '_posts' directory.

Our Jekyll project requires our posts to be in Markdown format but the articles we exported from Medium are in html format. We also need to get the images used in our Medium blogs and save it locally along with Jekyll [Front Matter](https://jekyllrb.com/docs/front-matter/). This process is complicated so we will be using a Python script to automate all the required process using a single line of command. 

Go ahead and clone the Python script from [here](https://github.com/sumeetbajra/medium-to-jekyll) inside our current project folder. You will execute the script using the python command. If you are using MacOS, Ubuntu or Debian, Python might already be installed on your system. If not, you can download Python from [here](https://www.python.org/downloads/). 

```
> cd beautiful-jekyll
> git clone https://github.com/sumeetbajra/medium-to-jekyll
```

If you followed our directory structure, you should have 'posts' folder containing all the Medium exported articles and 'medium-to-jekyll' folder containing our Python script all inside the 'beautiful-jekyll' project. Now go inside the 'medium-to-jekyll' folder and execute these commands.

```
// change directory
> cd medium-to-jekyll

// setup environment for the script
> pip install virtualenv
> virtualenv venv
> source venv/bin/activate
> pip install -r requirements.txt

// execute script
> python medium_to_jekyll.py --source ../posts --dest ../ --layout post --category blog

// exit the python virtual environment
> deactivate
```

Here, --source flag is the directory of the Medium exported 'posts' folder and -- dest flag is the root directory of our Jekyll project.

![](/img/screen-shot-2019-07-03-at-4.46.10-pm.png)

If you refresh the page at http://localhost:4000, you should see all your Medium articles.

Note: If you don't see the new posts, stop the Jekyll server and restart it again using `bundle exec jekyll serve` command.

![](/img/screen-shot-2019-07-03-at-5.04.57-pm.png)

After this, we won't be needing the 'posts' folder containing Medium exported articles and 'medium-to-jekyll' folder anymore. We can move them to separate directory outside our Jekyll project or delete them.

```
// go back to project root directory
> cd ..
> rm -rf posts
> rm -rf medium-to-jekyll
```

## 4. Integrate Netlify CMS

This is all great so far. We have our Medium articles exported and displayed in our own project. Next, we need to be able to add, update or delete blog posts from our own project. For that, we will be using [Netlify CMS](https://www.netlifycms.org/). 

Netlify CMS will allow us to manage our articles directly to Github repo. Thus, we will need to create a new repository for our blog. Let's go ahead and upload our 'beautiful-jekyll' to our own Github repository.

Create a new Github repository from [here](https://github.com/new). You can name the repository as "<github-username>.github.io". This will allow our site to be hosted via [Github pages](https://pages.github.com/).

_Note: Before you upload the project to Github, you might want to update the site's _config.yml file. It currently contains dummy information and you will want to replace it with your information before uploading it. You can make other changes too as you may feel necessary. Find more details on Jekyll config file_ [_here_](https://jekyllrb.com/docs/configuration/)_._

Let's initialize git in our folder and upload it to our newly created repository.

```
> cd beautiful-jekyll
> git init
> git add --all
> git commit -m "Initial commit"
> git remote add origin git@github.com:sumeetbajra/sumeetbajra.github.io.git
> git push -u origin master
```

After it's done, give it a couple of minutes for Github pages to deploy your app and you should see the blog up and running at https://<github-username>.github.io if you are using Github pages. In my case, the site is available at [https://sumeetbajra.github.io](https://sumeetbajra.github.io/)

**Creating Github OAuth app**

Now we have our Github repository ready, let's go and integrate Netlify CMS. We will be using Github OAuth for authenticating users to our blog and thus we need to create new Github OAuth app. Head over to [Settings](https://github.com/settings/developers) page or you can directly click here <https://github.com/settings/developers>

Click on "New OAuth App" and fill up the form. For 'Authorization callback URL', use "https://api.netlify.com/auth/done"

![](/img/screen-shot-2019-07-03-at-7.46.37-pm.png)

You will then get redirected to the OAuth app's page which contains ClientID and Secret. You will need this later.

**Creating Netlify app**

After creating the Github OAuth app, we need to create a Netlify app. Go to [https://app.netlify.com](https://app.netlify.com/) and login with your Github account. Once logged in, click on 'New Site from Git' and select Github.

![](/img/screen-shot-2019-07-03-at-7.52.56-pm.png)

Here, if you wish to host your Jekyll blog using Netlify too, you need to select the newly created blog repository. Otherwise you are free to select any repository you want. Even an empty repository like [this](https://github.com/sumeetbajra/netlify-backend) will work which I created for this example. In the next step, leave all the settings as it is and click 'Deploy site'. 

_If you are not hosting the Jekyll project on Netlify, keep 'Build command' and 'Publish directory' empty._

Now we need to connect our Netlify app to Github OAuth. Head over to the Settings page -> Access control -> OAuth

![](/img/screen-shot-2019-07-03-at-8.01.21-pm.png)

![](/img/screen-shot-2019-07-03-at-8.02.51-pm.png)

Click on Install provider and select Github as provider. Copy the Client ID and Secret from the Github OAuth app's page and paste them here.

Next, go to Settings -> Domain Management, click on 'Add custom domain'.

![](/img/screen-shot-2019-07-04-at-11.25.24-am.png)

Add your domain name <github-username>.github.io and click 'Verify' and then click 'Yes, add domain' if prompted with new message.

![](/img/screen-shot-2019-07-04-at-11.26.13-am.png)

Note your Netlify app's name. In my case, it is 'practical-shaw-b147f7'.

\---

Now let's go back to our project. We need to create a new folder inside our blog called 'admin'. Inside it, create two files namely 'index.html' and 'config.yml'. This 'config.yml' file is different from '_config.yml' file that is in the root folder. The content for each file is provided below.

<script src="https://gist.github.com/sumeetbajra/410df28a8afb7e68192852877393af8c.js"></script>

<script src="https://gist.github.com/sumeetbajra/b4ed7cd7af203d206eba1cb453a2aedb.js"></script>

In the config.yml file, replace 'repo' with your Github repo and 'site_domain' with your Netlify app's domain (<your-netlify-app-name>.netlify.com). 

If you want to know more about what each attribute in the config.yml file does you can check it out [here](https://www.netlifycms.org/docs/add-to-your-site/#configuration). 

Save the changes and then push them to Github. We are done now. If you head over to https://<github-username>.github.io/admin, you should see Netlify CMS's login page like this.

![](/img/screencapture-sumeetbajra-github-io-admin-2019-07-04-11_33_52.png)

_Note: It might take couple of minutes for Github pages to update._

Go ahead and login with your Github account and you will be redirected to the CMS's dashboard which contains all your posts. You can manage your blog posts from here.

![](/img/screencapture-localhost-4000-admin-2019-07-03-17_51_55.png)

## Using Custom domain name (Optional)

If you wish to use custom domain, you can either

1. Host the site in Netlify (which I mentioned briefly above under 'Creating Netlify app') and set your custom domain as primary domain in Netlify dashboard's Domain management. Then simply log in to the account you have with your DNS provider, and add a CNAME record pointing to <netlify-app-name>.netlify.com. 
2. Continue hosting the site in Github pages and add CNAME record pointing to <github-username>.github.io with your DNS provider.

Obviously, these are just in regards to this tutorial. You have the flexibility to customize the project as much as you want and host them wherever you prefer to.

Thanks for following through. I hope you liked the article. If you want to read more of Botsplash team contributions, check out articles [here](https://botsplash.com/blog).

Want to learn more about Botsplash platform? Write to us [here](https://botsplash.com/support).
