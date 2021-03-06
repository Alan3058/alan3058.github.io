---
layout: post
title:  "How to create github blog"
categories: [other]
tags: [github,github io,blog]
fullview: false
---
# Remark
Because of the blog of github io is simple and easy, so I am planning use the github io to replace my blog site. Now write the article to record the step of crete github blog, For more infomation, as below.

# Steps

## A. Create a simple github blog
1. Create a repository on github
  Create a  new repository on github,and the repository's name have a fixed rule.	For example, `yourgithubname.github.io`. this step is simple.

2. To add the index page
  Create a index.html file on the repository ,and add `hello world` to the file. Now when you input `https://yourgithubname.github.io` in the browser, it show `hello world`.  In the end, congratulation to you.
  
3. Replace your blog theme
  * Find your repository and click on the `Settings` tab. 
  * Scroll down to the `Github Pages` section, click on the `Choose a theme` button.
  * Pick a theme by you like, and click `finish` button.

> tip:For more information, see: [github pages site](https://pages.github.com/)

## B. Use jekyll on the blog,and local develoment
1. Config Ruby environment and install jekyll.
  * Download [ruby installer with devkit 2.4.4.1](https://github-production-release-asset-2e65be.s3.amazonaws.com/78153411/7d4c46da-3361-11e8-9a54-4786b55e58c2?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20180419%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180419T070555Z&X-Amz-Expires=300&X-Amz-Signature=7b9be014f39adde2684e06ae7d24bbd01fbc109ad346c09af053ae832c72ac1f&X-Amz-SignedHeaders=host&actor_id=19320830&response-content-disposition=attachment%3B%20filename%3Drubyinstaller-devkit-2.4.4-1-x64.exe&response-content-type=application%2Foctet-stream)
  * Install the ruby, and config ruby environment variable.
  * Install jekyll. execute `gem install jekyll` command.

2. Create a blog by jekyll
  execute follow command.
  ```shell  
  $ jekyll new blog 
  ```
  Now, you will found a blog directory in current directory. and the blog directory contains some files and a _posts directory, the _posts directory is used place the article.
  > About more information of jekyll directory and files: see [jekyll directory](http://jekyllcn.com/docs/structure/).
  
3. Copy the files to repository.
  copy the files of blog to the repository directory and push it to the github repository.  
  
4. Add article  
  The new article must written the directory by named `_posts`. For more information about article, you can see: [jekyll article](http://jekyllcn.com/docs/posts/).

5. Start jekyll
  ```shell 
  $ jekyll serve  
  ```

> tip:For more information: see [jekyll site](http://jekyllcn.com/docs/quickstart/).   
> also see [jekyll/liquid api document](http://ju.outofmemory.cn/entry/149459)

## C. Use your domain on the blog (Not Must)
1. Add a domain resolve on your DNS provider.
2. Add CNAME file to your repository, and write the domain on it.
3. Add your custom to your Github Pages site.

Now, you can use your domain to access.

## D. Use Paginate in your site
1. At first, write it on the _config.yml file, as below
```
paginate: 5  
plugins:   
  - jekyll-paginate
```
  
2. add jekyll-paginate plugin on the Gemfile, as below
> gem "jekyll-paginate"

3. If needed, update name of the index.md file to index.html. and it is a requirement of  jekyll.

4. install jekyll-paginate plugin
> gem install jekyll-paginate

5. Update the content of index.html file(show the pagination article). For example,as below  
![page-article](/assets/resources/image/20180419/page-article.png)
 
6. add the pagination link infomation to index.html file. For example,as below  
![page-link](/assets/resources/image/20180419/page-link.png)

Now, to visit your site, you will look the pagination effect.
