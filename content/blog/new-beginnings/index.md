---
title: New Beginnings
date: "2020-02-24T13:40:32.169Z"
description: The first blog in my entire developer career. I will summarize the steps I have taken to set up this site using GatsbyJS.
---

Yep, I start blogging 😊, this is considered big jump for me in terms of my work and life attitude. I have been reluctant to express or share my experience and knowledge online in past 10 years as a developer. I think this needs to be changed, not only I want to get more involved in the online community, but also I think it is a good approach to memorize what I have learn at the time. The rest of article I will explain how to set up and deploy this simple blog using GatsbyJS and Netlify.

## What is GatsbyJS

From [GatsbyJS website](https://www.gatsbyjs.org/)

> Gatsby is a free and open source framework based on React that helps developers build blazing fast **websites** and **apps**

In a nutshell, GatsbyJS is a static site generator based on NodeJS, ReactJS and GraphQL.

## How to set up

Install [NodeJS](https://nodejs.org/) if not yet.

**install Gatsby CLI**

```
npm install -g gatsby-cli
```

**Create new blog site using gatsby-starter-blog starter**

```
gatsby new my-blog https://github.com/gatsbyjs/gatsby-starter-blog
```

run below commands and navigate to http://localhost:8000/, you should be able to see default site up and running

```
cd my-blog
gatsby develop
```

**Write new blog**

This project is set up to look for blog posts under the `content/blog` directory. Create a markdown file with `.md` extension and input below content.

```
---
title: My First Post
---

Hey, this is my awesome new blog!
```
