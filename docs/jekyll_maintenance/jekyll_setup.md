---
layout: default
title: Web Development
nav_order: 4
---

## Web Dev notes for Jekyll / just-the-docs

### Format Gemfile (only required for initialisation)

If no gemfile in root folder of repository directory, make one (no file extension)

```sh
touch Gemfile
```

Add following contents by using nano

```sh
sudo nano Gemfile
```

```sh
source "https://rubygems.org"
gemspec
gem "jekyll"
```

### Initialise Jekyll Environment

After chroot to repository folder, enter following in terminal:
```sh
gem install jekyll bundler
```
Then launch site using following - this allows to make changes on the fly (markdown rendered to html with every save of .md file)
```sh
bundle exec jekyll serve
```

Then visit wherever home directory has been config (from separate terminal)

```sh
firefox 127.0.0.1:4000
```



