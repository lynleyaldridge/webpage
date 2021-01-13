---
title: "Making this blog: A step-by-step guide"
author: Lynley Aldridge
date: '2021-01-08'
draft: TRUE
slug: making-this-blog
categories: []
tags: 
  - Tutorial
  - Rstats
  - website
subtitle: ''
summary: ''
authors: []
lastmod: '2021-01-08T14:52:46+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

In this post I talk through, step-by-step, the process I used to create this website using R Studio, GitHub, blogdown, Hugo and Netlify. There are already plenty of great resources online that will talk you through this process, but this is written by a relative novice to both R and GitHub, in the hopes that my experiences will be of use to others in similar situations. 

Here are some of the resources I found most useful:

* Alison Hill's [Up and Running with Blogdown](https://alison.rbind.io/post/2017-06-12-up-and-running-with-blogdown/). As tweeted below, Alison has recently written an updated post to reflect changes to blogdown and the Hugo Academic theme - now ["Wowchemy"](https://wowchemy.com/)):

{{<tweet 1344727166228058114 >}} 

* The slides from [Summer of blogdown](https://summer-of-blogdown.netlify.app/), and 
[Up and Running with blogdown: A workshop for the Portland R User Group](https://alison.rbind.io/slides/blogdown-workshop-slides.html), also by Alison Hill. 
* The reference guide [blogdown: Creating Websites with R Markdown](https://bookdown.org/yihui/blogdown/) by Yihui Xie, Amber Thomas and Alison Presmanes Hill (note that this is currently being updated due to changes to blogdown and Hugo)
* Annie Lyu's [Fun blogdown in R to design a personal website](https://annielyu.com/slides/isugg/blogdown_aa#6)
* Martin Frigaard's [How to build a website with blogdown in R](https://www.storybench.org/how-to-build-a-website-with-blogdown-in-r/)
* Emi Tanaka's [Getting down and up with blogging in R](https://rladies-melb-blogdown.netlify.app/#1)
* Jenny Bryan, the STAT 545 TAs, and Jim Hester's [Happy Git and GitHub for the useR](https://happygitwithr.com/), which held my hand through each of the initial steps I needed to do to get GitHub working, as a total novice.

These step-by-step guides were an essential reference for me, and were really good at pointing me to all the additional resources I needed to work through the process of setting up this blog. However, I did need to do a lot of preparatory work using these resources to understand what I was doing at each stage and why. More importantly, I had to identify what the options were that I was choosing between, in order to make this blog one that would fulfill the goals I had for it. So I’ve provided a more detailed description of my process below.

## Step 1: Initital confusion and roads not taken

First, let’s talk about the initial confusion I experienced when I sat down to figure out what exactly I wanted to do, and what I needed to learn or put in place to get to the end result that is this blog. I read through a lot of people's accounts of making websites for the first time using a variety of different tools, and took some notes. GitHub, GitHub Pages, Git Bash Shell, Hugo, Jekyll, Distill, blogdown, Netlify? What are all these things? I've never even heard of some of them before!!  More importantly, how do they relate to each other?  Do some of them cluster together? Are there decisions that I should make about which route I want to go down before I go any further?

I've created a glossary at the end of this blogpost with links and brief explanations for each of these tools. Here are some useful references I relied upon heavily during this process (or found later and realised how helpful they were):

* Ma&#235;lle Salmon's [Blogging with R Markdown](https://rmd-blogging-jozi.netlify.app/), which talks about options for using Distill firstly, and secondly options for using Hugo and blogdown
* Emily Zabor's [Creating websites in R](http://www.emilyzabor.com/tutorials/rmarkdown_websites_tutorial.html), which talks about using R Markdown and GitHub pages firstly, and later using blogdown for creating blogs
* Desir&#233;e De Leon and Alison Hill's [rstudio4edu: A Handbook for Teaching and Learning with R and RStudio](https://rstudio4edu.github.io/rstudio4edu-book/), a work in progress, which talks through several different options for sharing materials online, the pros and cons of each, and 'Cookbooks' and lots of useful best practice tips for working with these tools. See also Alison Hill and Desir&#233;e De Leon's [Sharing on Short Notice](https://alison.rbind.io/talk/2020-sharing-short-notice/) webinar.

At the end of this process, I chose to make this website using 
R Studio, GitHub, blogdown, Hugo and Netlify. I'll talk more about why and what that means later, but first, I wanted to point to some of the alternatives I considered, in case they are more suitable for any readers of this post. 

I'm sure there are even more options than I'm listing here. But after reading and re-reading a lot of different descriptions of setting up online blogs and portfolios alongside the above resources, here are the options I identified from my reading, in what I perceive as their increasing order of complexity:

### RPubs
Perhaps one of the quickest and easiest ways of hosting R Markdown output online - without attempting to create a whole website - is to publish R Markdown output directly to RPubs. Example from Emi Tanaka [here](https://rladies-melb-blogdown.netlify.app/#8). Check out the [MarkyMark](https://rladiessydney.org/courses/ryouwithme/04-markymark-1/) section of RYouWithMe (created by R-Ladies Sydney) for a beginners tutorial on creating your fist R Markdown document and publishing to RPubs.  

If you just want a way of hosting documents online with a minimum of fuss, and a smaller learning curve, this might meet your needs sufficiently. (I linked directly to my RPubs profile for my first few job applications, because I wanted to prioritise getting documents online for potential employers to view over learning to master GitHub and website development.)

### R Markdown document
Desir&#233;e De Leon and Alison Hill provide instructions for creating a R Markdown document and publishing it online in their [Cookbook: R Markdown](https://rstudio4edu.github.io/rstudio4edu-book/intro-doc.html) from the rstudio4edu Handbook.

### Postcards
Sean Kross has developed the [Postcards](https://github.com/seankross/postcards) package for developing a single page website (based on four templates) from a single R Markdown document.  

{{<tweet 1338624860231139328 >}} 

### R Markdown's site generator
For hosting more than a single document, R Markdown has a built-in simple site generator that can generate a website with multiple pages and a navigation bar to navigate between pages. A tutorial from Emily Zabor can be found [here](http://www.emilyzabor.com/tutorials/rmarkdown_websites_tutorial.html).  See also the [Cookbook: R Markdown Sites](https://rstudio4edu.github.io/rstudio4edu-book/intro-rmd.html) from the rstudio4edu Handbook and [R Markdown documentation](https://bookdown.org/yihui/rmarkdown/rmarkdown-site.html). Files used in sites created by R Markdown's site generator all have to sit within a single directory, so this is recommended only for small websites with a limited number of pages.  

Note that this process involves hosting the website using GitHub pages (https://pages.github.com), and you may come across discussion of Jekyll in this context - it's a static site generator like Hugo that it's recommended to switch off as part of this process.  

### Distill 
The [Distill for R Markdown package documentation](https://rstudio.github.io/distill/) explains how this package can be used to create simple [websites](https://rstudio.github.io/distill/website.html) and [blogs](https://rstudio.github.io/distill/blog.html) using just R Markdown. (The former is a collection of pages you can navigate between using a navigation bar at the top of the page, while the latter involves the creation of a home page listing all posts.) This documentation is itself a distill website! For additional examples and tutorials on building your own distill website, check out blog posts by [Lisa Lendway](https://lisalendway.netlify.app/posts/2020-12-09-buildingdistill/) and [Thomas Mock](https://themockup.blog/posts/2020-08-01-building-a-blog-with-distill/), and the [Cookbook: Distill Sites](https://rstudio4edu.github.io/rstudio4edu-book/intro-distill.html) from the rstudio4edu Handbook. 

For a greater understanding of the differences between Distill and blogdown/Hugo, I found Frie's step-by-step description of [Transitioning from blogdown to distill](https://frie.codes/posts/tricks-blogdown-to-distill/) useful.

### Combining Distill and Postcards
* Alison Hill's [M-F-E-O: postcards and distill](https://alison.rbind.io/post/2020-12-22-postcards-distill/) walks the reader through an example of how postcards and distill can be combined so that one of the postcards templates can be used as the home page of a distill website.

### Using GitHub, blogdown and Hugo, with Netlify to host
This is the process that [Alison Hill](https://alison.rbind.io/post/2017-06-12-up-and-running-with-blogdown/), [Kristy Robledo](https://kristyrobledo.netlify.app/post/creating-my-blogdown-blog/), [Rebecca Barter](http://www.rebeccabarter.com/blog/2020-02-03_blogger/) and others have used.

## Step 2: Decision points?

Reviewing the options above, I chose to pursue the option of using R Studio, GitHub, blogdown, Hugo and Netlify to build and deploy my website.

Full disclosure, I was tempted to try using Distill instead, and regretted not doing so on several occasions. Distill seemed easier to learn, and involved many fewer moving parts to try to get my head around. Overall, it seems worth reading more about Hugo and the potential difficulties involved in maintaining a Hugo website before choosing Hugo/blogdown over Distill:

* Ma&#235;lle Salmon's [What to know before you adopt Hugo/blogdown](https://masalmon.eu/2020/02/29/hugo-maintenance/)
* Alison Hill's [A Spoonful of Hugo: How much Hugo do I need to know?](https://alison.rbind.io/post/2020-12-12-how-much-hugo/)

It's also important to know that the documentation for blogdown is currently outdated, although this seems to be because a new release with exciting new features is currently in development.  

{{<tweet 1347230510720933893 >}} 

But looking through some example blogs, I found myself preferring the aesthetic choices that seemed possible with a blogdown/Hugo combination. In particular, I decided that I wanted to be able to link publications to projects in ways that the widgets from the academic theme from Hugo (now ["Wowchemy"](https://wowchemy.com/)) made possible. The die was cast when I read comments from [Thomas Mock](https://themockup.blog/posts/2020-08-01-building-a-blog-with-distill/) and [Ma&#235;lle Salmon](https://rmd-blogging-jozi.netlify.app/conclusion/slides/#/1) suggesting that websites created using Hugo were more customisable and flexible. 

One caveat to this is that it seems different Hugo themes work differently depending on how they are configured, and customizing one theme may limit your ability to simply convert to another theme (see [this discussion](https://bookdown.org/yihui/blogdown/other-themes.html) in the blogdown reference guide. Making sure you’re happy with your chosen theme before investing heavily in the development of your website (or at least thinking carefully about compatibility issues) seems wise.  

General pointers on choosing themes are given [in the blogdown reference guide](https://bookdown.org/yihui/blogdown/themes.html), which recommends checking the star rating of the theme's GitHub repository, as well as its activity level. In choosing between the available [themes for Hugo](https://themes.gohugo.io/), I looked at the aesthetics of a lot of existing sites, as well as the documentation available for each, the recency and frequency of updates, and how well they matched with my vision of what I wanted my website to look like. Before settling on the Academic theme - now ["Wowchemy"](https://wowchemy.com/), I looked at several alternatives:

* [Kristy Robledo](https://kristyrobledo.netlify.app/post/creating-my-blogdown-blog/) and [Rebecca Barter](http://www.rebeccabarter.com/blog/2020-02-03_blogger/) use [Future Imperfect](https://themes.gohugo.io/future-imperfect/)

* [Martin Frigaard's tidyverse.tips](https://www.tidyverse.tips/about/) uses  [devise](https://themes.gohugo.io/devise/)

* [Diego Usai](https://diegousai.io/2019/10/build-your-website-with-hugo-and-blogdown/) uses Casper 2.1.7, but warns that this theme may not be supported by its developer


## Step 3: Summary process

## Step 4: Getting acquainted with GitHub

## Step 5: Creating a new site using blogdown 

## Step 6: Making inital changes to configurations

## Step 7: Testing deployment (push to GitHub and deploy via Netlify)

## Step 8: Troubleshooting

### Accented characters

https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references

&#225;

### Footnotes

https://www.markdownguide.org/extended-syntax/#footnotes

I want to add a footnote after this sentence.[^1]

[^1]: This text will become the first footnote.

## Step 9: Settling in for the long haul - additional configuration changes

# Glossary

  [GitHub](https://guides.github.com/activities/hello-world/#what): "GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere."

  GitHub Pages: 

  Git Bash Shell:
  
  [Distill for R Markdown](https://rstudio.github.io/distill/): "Distill for R Markdown is a web publishing format optimized for scientific and technical communication. "
  
  [R Markdown](): ""
