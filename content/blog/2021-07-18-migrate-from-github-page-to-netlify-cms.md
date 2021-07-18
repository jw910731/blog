---
title: Migrate from Github Page to Netlify CMS
date: 2021-07-18T10:40:41.754Z
description: My blog migration from github pages to netlify cms with github action CI/CD
---
# Background

I was recently planning about migrate my blog from Github Pages to Netlify, due to the lack of online edit solution for Github Pages. I had tried using gitpod as my online editor but it didn't work out well. I need to wait for gitpod to build the docker image before entering the editor, and the edit experience is not that well on mobile devices.
Netlify offers Netlify CMS which I think it is worth to try after reading its document.

The migration at first is rather easy. The migration of static content is quite simple. The Netlify CMS was not that hard to setup with the guide of official document. At this moment, I had no idea I will be stuck at here for that long time. 

# Stuck

The Netlify CMS it self is almost working well. Every function work well except Show Preview Link. When saving drafts in editorial workflow, it should create or force push a Pull Request and wait for CD pipeline to run. If the result of deployment met a certain condition. The "View Preview" button shows up.
But the mysterious condition is not clearly stated in the document. I had no idea why I had already setup the CI pipeline but the View Preview button is not showing up.

First, I made a mistake that I was used to Github Pages' publish method. That is, push the compiled files to a certain branch and it simply serves the content in that branch. Netlify had it's own publish method. I need to integrate my old CI pipeline with it's CLI tool to publish my content to it. The official CLI wrapper of Github Action is too slow to startup. I finally decide to use [this](https://github.com/nwtgck/actions-netlify) Github Action. Which is written in Typescript and run much faster than official tool that need to wait for npm to install the CLI tool.

Secondly, I still had no idea with the unclarified condition which need to met in order for Netlify CMS to know that this is the build result of preview link. After searching for several github issue of Netlify CMS and posts on Netlify support forum. I found that in [this document](https://www.netlifycms.org/docs/github-backend/#specifying-a-status-for-deploy-previews). It check the "context" of the commit status whether containing keywords related to deployment preview. **What is keyword related to deployment preview?? Are you kidding me???** I found that the "context" of commit status is the text in check section of PR.

![](/img/gh-screenshot.png)

Which the `target_url` or the link pointed by "detail" button after the text need to be pointed to the preview link. 

Finally, I found a option in the same document, called `preview_context` which can change the keyword that triggers the preview link. I tried it with `github` backend and `git-gateway` backend which both backends are working fine with this option. (Note that I use Github as my Netlify Identity login method. I have not test without login with Github.)

# Summary

After setting `preview_context` and using the 3rd party Github Action script to deploy my content to Netlify. The CMS is finally showing preview link to me automatically. Though the check preview button is acting weird. It's still better than writing posts in gitpod.
The codes are public viewable on Github in case you need it. [Link](https://github.com/jw910731/blog)
Also, this blog post is written in the Netlify CMS. The edit experience is quite nice to me.