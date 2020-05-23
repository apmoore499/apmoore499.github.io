## Github doc


## Setting up

Open up ```_config.yml``` and append following lines:

```sh
 url: <home url >
 baseurl: /<repository name>
```

## Cloning repo

Get https link by perusing repository in browser. Click on button that says 'Clone or download' and copy https link

Then chdir to working folder on local machine and clone
```sh
git clone <github_https_link>
> Cloning into `<repo_name>`...
> remote: Counting objects: 10, done.
> remote: Compressing objects: 100% (8/8), done.
> remove: Total 10 (delta 1), reused 10 (delta 1)
> Unpacking objects: 100% (10/10), done.
```


You should be able to force your local revision to the remote repo by using

```sh
git push -f <remote> <branch>
```

(e.g. git push -f origin master). Leaving off < remote > and < branch > will force push all local branches that have set --set-upstream.


## Initialising repository

Initialise repository with


```sh
git init
```

## Adding to old repository

In Terminal, add the URL for the remote repository where your local repostory will be pushed.

```sh
git remote add origin <remote repository URL>
```

git remote add origin https://github.com/apmoore499/apmoore499.github.io.git



apmoore499/apmoore499.github.io



## Github can't host certain versionf of jekyll so use this code in Gemfile, also delete Gemfile.lock


```sh
gem "jekyll", "~> 3.8.5"
gem "github-pages","~> 202" , group: :jekyll_plugins
gem "just-the-docs"
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.11.0"
end
```

## If you need to add dependencies, you must do so in the package.json file, eg


```sh
"dependencies": {
  "minimist": ">=0.2.1"
}
```

original package.json:

```sh
{
  "name": "just-the-docs",
  "version": "0.2.8",
  "description": "A modern Jekyll theme for documentation",
  "repository": "pmarsceill/just-the-docs",
  "license": "MIT",
  "bugs": "https://github.com/pmarsceill/just-the-docs/issues",
  "devDependencies": {
    "@primer/css": "^14.3.0",
    "prettier": "^2.0.5",
    "stylelint": "^13.3.3",
    "stylelint-config-prettier": "^8.0.1",
    "stylelint-config-primer": "^9.0.0",
    "stylelint-prettier": "^1.1.2",
    "stylelint-selector-no-utility": "^4.0.0"
  },
  "scripts": {
    "test": "stylelint '**/*.scss'",
    "format": "prettier --write '**/*.{scss,js,json}'",
    "stylelint-check": "stylelint-config-prettier-check"
  },
  "dependencies": {} 
}


```

updated package.json (note down the bottom dependencies list changed)

```sh
{
  "name": "just-the-docs",
  "version": "0.2.8",
  "description": "A modern Jekyll theme for documentation",
  "repository": "pmarsceill/just-the-docs",
  "license": "MIT",
  "bugs": "https://github.com/pmarsceill/just-the-docs/issues",
  "devDependencies": {
    "@primer/css": "^14.3.0",
    "prettier": "^2.0.5",
    "stylelint": "^13.3.3",
    "stylelint-config-prettier": "^8.0.1",
    "stylelint-config-primer": "^9.0.0",
    "stylelint-prettier": "^1.1.2",
    "stylelint-selector-no-utility": "^4.0.0"
  },
  "scripts": {
    "test": "stylelint '**/*.scss'",
    "format": "prettier --write '**/*.{scss,js,json}'",
    "stylelint-check": "stylelint-config-prettier-check"
  },
  "dependencies": {
  "minimist": ">=0.2.1"
  } 
}


```
