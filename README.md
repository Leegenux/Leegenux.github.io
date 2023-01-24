# Introduction

This project serves as a personal repository of notes, for what I've learnt and I am thinking about.

To add new notes, first switch to the branch `blogging` and write new posts in `source` folder. 
This site is powered by HEXO blog framework.

# Deployment

First install the `hexo` package:
```
npm install hexo-cli
```
You can then execute the command using 
`npx hexo` or `node_modules/.bin/hexo`

The static files are generated with 
```
hexo generate
```
and automatic deployed with
```
hexo deploy
```

There are other combos, e.g., `hexo generate --deploy`, that can be used to simplify the pipeline.
