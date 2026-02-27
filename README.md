[![Netlify Status](https://api.netlify.com/api/v1/badges/2c75d4a8-9cd9-42b8-bff8-8bb9e0cbfd76/deploy-status)](https://app.netlify.com/sites/micromata-labs/deploys)

# Micromata Labs - Jekyll theme

Based on _Desk Theme_ by [Unbound Studio](https://unbound.studio).



## Installation



### Prerequisites

1. Ruby 3.0.x
  - I highly recommend you install ruby using [rvm](https://rvm.io/)!
2. Bundler & Jekyll
  - Can be installed using: `gem install bundler jekyll`
3. Node >= 14 (or 12, it's not *that* important for this)

### Dependencies

This project uses two dependency management systems:
- Bundler for Jekyll plugins
- NPM for frontend dependencies



#### Bundler

Bundler dependencies can be installed by running:
```shell
bundle install
```

On macOS, there might be an error when  building the `eventmachine` dependency because of openssl. 
This can be solved by installing openssl and then pointing bundler to the correct location:

```shell
brew install openssl
gem install eventmachine -v '1.2.7' --source 'https://rubygems.org/' -- --with-cppflags=-I/usr/local/opt/openssl/include
```

⚠️ On a macOS with **Apple Silicon**, use:

```shell
brew install openssl
gem install eventmachine -- --with-openssl-dir=/opt/homebrew/Cellar/openssl@1.1/1.1.1m/
```
Do not forget to check the version of of your openssl, e.g. */openssl@1.1/1.1.1**m**/* !



#### NPM

NPM dependencies can be installed by running:
```shell
npm install
```

Run the following to generate your site:
```shell
npm run build
```


### Development

To develop the site using livereload and extensive error reports, run the following:
```shell
bundle exec jekyll serve --livereload --trace
```



## Site structure

- **commmunity**
  - *sec-meetup-kassel.md*
  - *etc.*
- **f-und-e**
  - *profi.md*
  - *vaminap.md*
  - *etc.*
- **_docs**
  - *baumeister.md*
  - *projektforge.md*
  - *java-api-for-kml.md*
  - *etc.*
- **publikationen**
  - *awesome-lists.md*
  - *micromata-tv.md*
  - *etc.*
- **_posts**
  - *2021-02-12-your-blog-post.md*
  - *2021-02-12-my-blog-post.md*
  - *etc.*

## Adding author details

Add new authors details in `_config.yml`:
```yaml
# Author settings, displayed on post and doc pages if front matter references author name e.g. author: ckuehl
authors:
  mkuehnel:
    name:         Michael Kühnel
    title:        Frontend Developer
    shortdesc:    Michael ist seit Netscape 4.7.8 im Geschäft und freut sich, dass dank Webstandards seitdem alles besser wird. Folge Michael bei <a href="https://twitter.com/mkuehnel">Twitter</a> oder <a href="https://github.com/mischah">GitHub</a>.
    avatar:       avatar-mkuehnel.jpg
    email:        ckuehl(at)micromata.de
```

- Add a reference author name, e.g. *ckuehl*

- `name:` Add full name, e.g. *Christian Kuehl*

- `title:` Add a job title, e.g. *Developer*

- `shortdesc:` Add a short description (optional); you can use inline html to add links etc.

- `avatar:` Add an avatar image (optional), upload the image to `/uploads`. If you do not add an avatar, a default avatar will be used.

- `email:` Add an email adress (optional), e.g. *ckuehl(at)micromata.de*

  

## Create a new blog article

To create a new post, you can create a new markdown file inside the `_posts` directory by following the recommended file naming format:
```
YEAR-MONTH-DAY-title.MARKUP
```
Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. For example, the following are examples of valid post filenames:

```
2021-01-12-micromata-labs-is-awesome.md
2020-09-04-how-to-write-a-blog.md
```

Post requires front matter, everything in between the first and second --- are part of the YAML Front Matter, and everything after the second --- will be rendered with Markdown and show up as “Content”.

The following is a post file with different configurations you can add as example:

```yaml
---
title: Your Blog Post Title…  # The full title of your post
author: atippel               # Authors reference name (see _config.yml)
categories: [Best Practices]  # Blog post category, e.g. "Best Practices", "Quick Tips" (these are the only categories at the moment)
tags: [featured, java, maven] # Group similar topics, e.g. "java". Use "featured", if you want to show up the new blog post on the home page.
shortdesc: Versionskennzeichen automatisch und zentral vergeben. # Show as a short description of your blog post on the home page
---

# Here comes your blog post!
Lorem ipsum...
```

To keep things more organized, add post images to `/assets/posts/` directory, and add theme images to `/assets/img/` directory.



## Creating a project article

Step 1: Create new project article (for Micromata Open Source projects) in `_docs` folder, similar to creating posts, but with following front matter settings (for example, the filename for the article would be `check-packages.md`):

```yml
---
title: Check Packages
subtitle: This is your optional doc subtitle
image: logo-baumeister.jpg # Add your project logo or image at the top of the page (optional)
tags: [npm, node, cli] # Group similar topics; adds a list of similar projects at page bottom (optional)
author: ckuehl
---
```
Step 2: Add your new document title to the header navigation.
Header navigation for project posts can be edited in `_data/navigation_header.yml`:

```yml
- title: Open Source
    dropdown:

      - title: Baumeister
        url: /docs/baumeister/

      - title: Check Packages				# Name for navigation drop down
        url: /docs/check-packages/	# file name, without file extension

      - title: Java API for KML
        url: /docs/java-api-for-kml/

      - title: Home Made Projects
        url: /docs/home-made-projekte/
```

Step 3: Add your new document title to the sidebar navigation.
Sidebar navigation for project post can be edited in `_data/navigation_docs.yml`:

```yml
- title: Micromata Projekte # Section title
  docs:
  - baumeister
  - check-packages          # file name from _docs folder, without file extension
  - java-api-for-kml
  - home-made-projekte
```



## Creating entries for home made project

Create new  entries in `_data/navigation_boxes_homemade.yml` :

```yml
- title: Rickrollify
  desc: A Chrome extension which rickrollifies every link on every website – Psst!
  url: https://github.com/bullgit/Rickrollify
```

*Note: Home made project are just a list, they do not have their own project page.*

##  Project boxes section

These are the project teaser on the home page. Project boxes are added in `_data/navigation_boxes.yml`, e.g.:

```yml
- title: Check Packages
  desc: Check packages ermöglicht die Überprüfung von npm Dependencies eines Projektes via whitelisting (oder blacklisting).
  icon:								# not used
  doc: check-packages	# file name from _docs folder, without file extension

- title: ProjectForge
  desc: Eine webbasiertes Tool zur Unterstützung aller Verwaltungsaufgaben im Projektmanagement und der Unternehmensorganisation.
  icon:								# not used
  doc: projectforge
```




Full documentation can be found here: [https://jekyll-theme-docs.netlify.com/docs/installation/](https://jekyll-theme-docs.netlify.com/docs/installation/).
