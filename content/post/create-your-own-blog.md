---
title: "Create your own blog in less than 1 hour for free using Hugo"
date: 2019-06-05T01:22:42+02:00
author: "Bhanu Chaddha"
linktitle: Create your own blog in less than 1 hour for free using Hugo
next: /tutorials/github-pages-blog
prev: /tutorials/automated-deployments
weight: 3
authorAvatar: image/logo.svg
image: image/3/selfBlog.png
tags : [
    "hugo",
    "frontend",
    "blog",
    "tutorial",
]
categories : [
    "tutorial"
]
draft: false
---

In the technical age, publishing articles online is a great way to express yourself. It could be of any form like tutorial, blogs, expessing opinion and much more.

It is almost defacto to use [Medium]() for bloging now a days. It provide you many benfits like :

* Only focus on content. Formatting is handled by Medium platform itself
* SEO (Search Engin Optimization) to promote your article in google search
* Easy interface to write article in browser itself
* And many more like support for images, videos and code reference using [Gist]()

**Then Why shoud you have your own blog?**

Having your own blog provide you the chance to add one more aspact of social presence. You can control the content as you want and decide whether your content should be paid or open to all. You can use this blog/website as your portfolio if you want.

## How did I do it?

After wring couple of [articles]() on medium. I though of setting up my own blog website. I looking into couple of options like :

* [WordPress]()
* [Wix]()
* [Ghost]()

These platform provide you end to end solution to setup your own website. These are either paid or partialy free. But as the tiltle of the article suggest, I wanted to do it for free. And I bet you are also here for the same.

Apart from being free I also wanted the blog setup to be mimimal. That means I only wanted to manage the content of the blog and not the look and feel or formatting. So that I can save myself from the hassel of managing UX, Styling, Hosting, Database and other elements of website management. It was inpired from the experience of writing a blog on Medium. Just write the content at correct location and **Boom!!**. 

I was aware that github allow you to have project website hosting for free if this is a open source project. You can read about it [here]().

And because github website only support Static Website( No Server, No Databse), I wanted a way to generate Static website with evolving content.

## Here comes the Hugo

Which searching such tool I came accross an awesome to framwork called [Hugo](). Quoting from the offical website 

>Hugo is one of the most popular open-source static site generators. With its amazing speed and flexibility, Hugo makes building websites fun again.

Hugo can be used for complete Content Management where content can be provided in many ways. It has lot of feature for customization and organizing the content. For the brevity of the blog I would not go into the deatails of that. Remember we  have less than 1 hour.

## Shut Up!! and Show me the code

Ok Ok.. Lets do that

We gonna need to do exactly 5 steps to set up our website:

1. Download and Set up Hugo
2. Create Project
3. Write Your Blog
4. Set up git repositories
5. Push your content and Done!!


### **Download and Set up Hugo**

You can follow the [setup instrutions](https://gohugo.io/getting-started/installing/) from [Hugo's official website](). They have instruction for each platform. I will be using Mac in this tutorial.

After set up completed . You can check if you have Hugo correctly installe dusing below command. It should print current installed verions of Hugo 

```
hugo version                                                                                                            
Hugo Static Site Generator v0.55.6/extended darwin/amd64 BuildDate: unknown
```

### **Create Project**

Go to your command line and run below commands
```
hugo new site demo-blog                                                                                                      
```
You would see below result. Your basic hugo project is ready now.

```
Congratulations! Your new Hugo site is created in /Users/bhanuchaddha/Documents/project/demo-blog.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

Hugo at core is static site generator. But it does not govern look and feel of your website. You can create different type of website using Hugo like Lending Page, Blog, Company website, Documentation, Portfolio, Resume, Podcast. You can decide your own layout and presentation and organize those yourself. But thanfully you dont have to create everything from the scrach. You can use any of the exiting available [themes](https://themes.gohugo.io/) and customize it.

For our blog website we would use [Casper Two](https://themes.gohugo.io/hugo-casper-two/) theme.

Type below commands to install theme.

```
 ~/Documents/project cd demo-blog

 # Initialize git repository    
 ~/Documents/project/demo-blog  git init                                                                                
Initialized empty Git repository in /Users/bhanuchaddha/Documents/project/demo-blog/.git/
 ```

 ```
 #Install Theme
 ~/Documents/project/demo-blog cd themes 
 ~/Documents/project/demo-blog/themes git clone https://github.com/eueung/hugo-casper-two.git casper-two

```
You will see below output

```
Cloning into 'casper-two'...
remote: Enumerating objects: 522, done.
remote: Total 522 (delta 0), reused 0 (delta 0), pack-reused 522
Receiving objects: 100% (522/522), 3.96 MiB | 5.28 MiB/s, done.
Resolving deltas: 100% (233/233), done.

```

Open your project in code editor of your choice. I would be using [VSCode]() in this tutorial. Every theme require some configuration changes. 

Replace content on `config.toml` file with below 

```
baseurl         = ""
theme           = "casper-two"
languageCode    = "en-US"
disqusShortname = ""
paginate        = 6
#SectionPagesMenu = "main"

[params]
  title       = "<Your Name>'s blog"
  subtitle    = "Blog | Tutorial"

  cover       = ""
  description = "<Your Name>'s blog"
  metaDescription = ""
  googleAnalytics = ""
  customCSS = []
  RSSLink = ""

  twitterName = "<Your twitter username>"
  fbName = "<Your fb username>"
  githubName = "<Your github username>"
  linkedinName = "<Your linkedin username>"
  # mediumName = "<Your medium username>" // disabled medium link

  logo = ""
  orgName = "EM"
  orgWebsite = ""
  orgDescription = ""

  author = "EM"
  authorAvatar = ""
  authorLocation = "<Your Location>"
  authorWebsite = ""
  authorDescription = ""

  pageNotFoundTitle = "404 - Page not found"

  #casper or caspertwo
  singleViewStyle = "casper"

[permalinks]
  post = "/:slug/"

[[menu.main]]
  name = "Blog"
  url = "/"
  weight = 200

[[menu.main]]
  name = "Tutorial"
  url = "/categories/tutorial/"
  weight = 5
```

now go to you root directory `demo-blog` and run below command

```
hugo server
```

This will start a local server at `http://localhost:1313`. If you go this url using any browser, you should see below

![Initial Blog Website](/image/3/initialBlog.png)

Wow!! your blog website is almost setup. The only thhing missing is your first blog. Which we would write in next section.

