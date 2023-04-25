+++
categories = ["howto"]
draft = false
date = "2017-01-06T17:15:03Z"
title = "How to Hugo"

+++

Here's how to publish new posts on this blog using Hugo.

* Install hugo (via apt or from https://github.com/spf13/hugo/releases)

```bash
apt install hugo
```

* Setup the repo

```bash
git clone -b source git@github.com:brk3/brk3.github.io.git
```

* Create a new post

```bash
hugo new post/how-to-hugo.md
<edit>
git commit -av
```

* Republish the site

```bash
hugo
git checkout master
cp -r public/* .  # Note, this will not show changes in the index!
git add .
git commit -am "Regenerate Site"
git push --all
```
