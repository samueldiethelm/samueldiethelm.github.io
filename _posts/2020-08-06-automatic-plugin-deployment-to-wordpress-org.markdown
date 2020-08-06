---
layout: post
categories: git
title: Automatic Plugin Deployment to WordPress.org with Github actions
tags: [wordpress, github, actions, svn]
---

Git is an immensly popular and useful tool among developers. There are also other version tracking systems such as SVN, which you will need to work with if you plan to upload an opensource plugin to wordpress.org. I personally don't like SVN too much and that is why I looked up how to automate code delivery to SVN from within Github Actions. This could also be done easily with any other automation service but I decided to give Github Actions a try.

<!--more-->

After checking several approaches I have simplified it to the following: 

Create a file `.github/workflows/deploy.yml` with the following content:
```
name: Deploy to WordPress.org
on:
  push:
    tags:
    - "*.*"
jobs:
  tag:
    name: New tag
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: WordPress Plugin Deploy
      uses: 10up/action-wordpress-plugin-deploy@stable
      env:
        SVN_PASSWORD: ${{ secrets.SVN_PASSWORD }}
        SVN_USERNAME: ${{ secrets.SVN_USERNAME }}
        SLUG: my-wp-plugin-slug
```

The above will tell github to upload the code to WordPress' SVN whenever a tag is pushed with the format `*.*`, such as `1.1`, `4.2`, etc. There is also an optional `SLUG` parameter to tell the exact WordPress plugin slug, however this can be deleted if it matches exactly the github repository slug.

Additionally I created a file in the root plugin directory: `.gitattributes` with the following content: 
```
# Directories
/.github export-ignore

# Files
/.gitattributes export-ignore
/.gitignore export-ignore
```
This should tell github not to export those files/directories to WordPress plugin directory and thus avoiding clutter. Right now I have not added a readme.md file, but if so, it could also be added to this file.

In the first file that was created, there were two _secrets_ in the code, which are the wordpress.org username and password. These need to be set in our github repository settings, under "Secrets" section with names `SVN_USERNAME` and `SVN_PASSWORD` respectively.

Once the above is all done, we should be able to push a new tag to github and github will take care of automatically uploading plugin changes to wordpress.org.

## Resources:
* [10up - action-wordpress-plugin-deploy](https://github.com/10up/action-wordpress-plugin-deploy){:target="_blank"}
* [Github Actions](https://github.com/features/actions){:target="_blank"}
