---
title: 搭建个人博客-git and hexo
date: 2022-01-10 16:02:58
tags: git, hexo
---

## Env Config

- Git/node/hexo

install hexo-cli globally

```bash
npm install -g hexo-cli
```

hexo command

```javascript
npm install hexo -g // install hexo globally
npm update hexo -g // update hexo verison
hexo init // init blog
// cmd abbreviation 
hexo n "my blog" == hexo new 'my blog' // create a new article
hexo g == hexo generate
hexo s == hexo server # start a server to preview, can hot update
hexo d == hexo deploy
// diff cmd for server
hexo server // hot update
hexo server -s // static server
hexo server -p 5000 // update port 
hexo server -i [ip address] // custom server ip
hexo clean # remove cache
```

## Progress

- create a new git repository  name：[username].github.io
- Create blog project by hexo

```bash
hexo init blog
```

- Create a new article and preview

```bash
hexo n "new blog"
hexo g
hexo s // open localhost:4000 in browser
```

- publish blog to git 

```blog/_config.yml : site config file```

```blog/themes/_config.yml: theme config file```

connect blog and git repo: config cmd ```hexo d``` to let hexo know where you want deploy your blog

```bash
vim blog/_config.yml
# modify last item deploy
deploy:
  type: 'git'
  repo: 'https://github.com/username/username.github.io.git'
  branch: 'master'
```

Install git deploy plugin

```bash
npm i hexo-deployer-git -D
```

Deploy blog to git

```bash
hexo clean
hexo g
hexo d
```

- change theme
  - you can choose a theme you like in site https://hexo.io/themes/
  - follow the guide to use the theme
  - below is a example for use theme next
  - Download theme to blog/themes/[themename]
  - Below is a sample for themes/hexo-theme-sky https://github.com/iJinxin/hexo-theme-sky

```bash
cd blog
git clone git@github.com:iJinxin/hexo-theme-sky.git themes/sky // download hexo-theme-sky to blog/themes/sky/
```

- change site config file: blog/ _config.yml

```bash
theme: sky
```

install dependency

```bash
npm install
npm install hexo-renderer-scss --save 
```

Note: install hex-renderer-scss error

Error Reason: node-sass install error

Resolve: https://github.com/lmk123/blog/issues/28
