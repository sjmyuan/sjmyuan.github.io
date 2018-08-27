---
title: How To Set Up A Personal Blog
tags:
  - Tools
---

Blog is a good place to share your knowledge, enhance your skills. It's pretty cool to have your own blog website.

A good blog should have the following features:

* Markdown

  Markdown is a popular tool to compose document, its syntax is clear and very easy to use. In this chapter, we will use [Kramdown](https://kramdown.gettalong.org/)

* Profile

  Profile is very important, people can know who you are and contact you.

* Index

  Good index can give better experience to your reader

* SEO

  To let more people know your blog, you may want people can search your blog on Google, Bing or other seach engine, so you need a proper SEO setting for them.

* Monitoring

  A good monitoring of blog can let us know how many people visited our blog, which blog is most popular, how much money our blog worth.

* Comments

  Comments is a good place to disucss the content of blog with other people, you can get better feedback and do it better and better.

Now let's see how to set up a blog site step by step

# Infrastructure

We will use [GitHub Pages](https://pages.github.com/) to host our blog, the first step we need to do is creating a new repo named **\<name\>.github.io **, such as

![](https://ws4.sinaimg.cn/large/006tNbRwly1fundsmy9vpj30oi09w75g.jpg)



Then enable the GitHub Pages on this repo, here we select the master branch as the source of blog

![](https://ws1.sinaimg.cn/large/006tNbRwly1funeaid7llj30xy0pu781.jpg)

After these two steps, you can visit your blog from **http://\<name\>.github.io**, of course you will find it's an empty page.

Now let's see how to set up the basic page of blog. the backend of GitHub Pages is [jekyell](https://jekyllrb.com/), the directory structure of jekyell project looks like this

```sh
|--_include    # header,footer,scripts
|--_posts      # blogs
|--_data       # navigation settings
|--_page       # custom pages
|--_config.yml # main configuration
|--Gemfile     # dependencies
|--index.html  # main page
```

Currently we will just need **_config.yml**, **Gemfile** and **index.html**, they will give us one simple blog page.

Although GitHub Pages already integrated some [themes](https://pages.github.com/themes/), but here we will use one thrid party theme called [minimal-mistakes](https://github.com/mmistakes/minimal-mistakes), it has a very good [document](https://mmistakes.github.io/minimal-mistakes/) and support kramdown

For Gemfile, we need following dependencies

```ruby
source 'https://rubygems.org'

gem "github-pages", group: :jekyll_plugins # this requied by GitHub Pages
gem "jekyll-compose", group: :jekyll_plugins # this lib supply lots of useful utility methods
```

For index.html, we need following content

```html
---
layout:home
---
```

For **_config.yml**, we need a template from [here](https://github.com/mmistakes/minimal-mistakes/blob/master/\_config.yml), read this template carefully, it contains everything we need to know. after download this basic configuration, we need to change the theme as following

```yaml
#theme                    : "minimal-mistakes-jekyll"
remote_theme             : "mmistakes/minimal-mistakes"
```

To have a default layout for the home page and post page, we can add the following settings to **_config.yml**

```yaml
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      # show your profile
      author_profile: true 
      # show the read time estimation
      read_time: true 
      # show the comments
      comments: true
      # show the table of content
      toc: true
      # show the share button
      share: true
      # show the related blog
      related: true
  # _pages
  - scope:
      path: ""
      type: pages
    values:
      layout: single
      author_profile: true
```

Now push these three file to your blog repo, then visit **http://\<name\>.github.io**, you should see a basic blog page.

After this section, we have a basic infrastructure, We will refine it in the following sections

# Profile

minimal-mistakes have strong support for profile, you can set the name of your blog, upload your portrait, add the link of your GitHub, LinkIn etc. everything you need to do is changing the file **_config.yml**, here is an example

```yml
title                    : "Demo"
title_separator          : "-"
name                     : "Demo Blog"
description              : "Enjoy Your Life"
url                      : "http://example.com"
repository               : "example/example.github.io" 
author:
  name             : "Tom"
  avatar           : "/images/bio-photo.jpeg"
  bio              : "Everything is good"
  location         : "Earth"
  email            : example@gmail.com
  github           : example
  linkedin         : "example-1111"
```

# Index

We usually use tag, category or year to group our blogs, so it's a good experience for reader to list your blogs by tag, category or year. To do this, we need to add some custom pages in folder **_page** and add links in navigation to link to these pages in folder **_data**.

For custom pages, you can refer these example

* [Year Page](https://github.com/sjmyuan/sjmyuan.github.io/blob/master/_pages/year-archive.md)
* [Tag Page](https://github.com/sjmyuan/sjmyuan.github.io/blob/master/_pages/tag-archive.md)
* [Category Page](https://github.com/sjmyuan/sjmyuan.github.io/blob/master/_pages/category-archive.md)

For navigation, you can refer this example

* [Navigation](https://github.com/sjmyuan/sjmyuan.github.io/blob/master/_data/navigation.yml)

# SEO

To have a better [SEO](https://en.wikipedia.org/wiki/Search_engine_optimization), you should [verify your website](https://support.google.com/webmasters/answer/35179?hl=en) using Meta tag method and put your verification code in **_config.yml**, here we just give an example of Google

```yaml
google_site_verification : "<verification id>"
```

# Monitoring

[Google Analytics](https://analytics.google.com/analytics/web/provision/?authuser=0#provision/SignUp/) is a good tool to monitor your website, you just need to register you website and get the tracking id, then put it in the **_config.yml**

```yaml
analytics:
  provider               : "google"
  google:
    tracking_id          : "<tracking id>"
```

Then you can see a beautiful dashboard on Google Analytics for your website, looks like this

![](https://ws3.sinaimg.cn/large/006tNbRwly1fuogg29ozkj31jw0uu79j.jpg)



# Comments

[Disqus](https://disqus.com/) is a very popular tool to manage your comments, you can register your website and get a short name of disqus, then add it in **_config.yml** like this

```yaml
comments:
  provider               : "disqus"
  disqus:
    shortname            : "<shortname>"

```

Then you can see the comments on your blog like this

![](https://ws4.sinaimg.cn/large/006tNbRwly1fuogn2ytt3j313q0kuac4.jpg)

# Summary

After these settings, you can begin to compose your post. the name of post should follow pattern **YYYY-MM-dd-\<name\>.md**, to simplify the workflow we can use the utility method of **jekyll-compose** like this

* Genrate a draft post

  ```sh
  $ bundle exec rake jekyll draft <post name>
  ```

  The **\<post name\>.md** file will be created in folder **_drafts**

* Add the title, layout, tags and categories in the [Front Matter](https://jekyllrb.com/docs/frontmatter/) of posts

  ```yaml
  ---
  title: Demo
  tags:
    - Blog
  categories:
    - Tutorial
  ---
  ```

  Note that we already added the default settings of post in the `Infrastructure`section, the settings in posts will overwrite the default one.

* Review the post in the local environment

  ```sh
  $ bundle exec rake jekyll serve --drafts
  ```

  You can access your blog from **http://localhost:4000**

* Publish this post

  ```sh
  $ bundle exec rake jekyll publish _drafts/<post name>
  ```

  This command will move the post to folder **_posts** and rename it to the pattern **YYYY-MM-dd-\<name\>.md**

* Finally you can commit this post to your git repo and access it online

minimal-mistakes also support lots of other features, such as search, pagination and themes. you can go through its document and try them on your website. Hope this blog is useful for you.