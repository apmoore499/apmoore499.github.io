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

git remote add origin <remote repository URL>
