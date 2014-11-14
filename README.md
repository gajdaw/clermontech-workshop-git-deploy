Clermont'ech Workshop Git
=========================


## 1. Deploy Using Git Push/Hook

We need two separate folders to simulate two different servers:

    mkdir {local,remote}-server

In the first folder, let's create an app:

    cd local-server
    mkdir app
    cd !$
    git init
    echo "Hello, World" > index.html
    git add index.html
    git commit -m "Initial commit"

Let's add a new remote that points to the `production` server:

    git remote add production /absolute/path/to/remote-server/www

Now, we are going to configure the remote server.

    cd ../../remote-server
    mkdir www
    cd !$
    git init

Let's edit the `.git/config` file, adding the content below to allow push on the
current branch:

```
[receive]
    denyCurrentBranch = false
```

Now, let's create a `post-receive` hook with the following content:

``` bash
#!/usr/bin/env bash

LOGFILE="/tmp/deploy-app.log"

function notify()
{
    echo "Hi! I am your Git deployer, and I am deploying branch \"$1\" right now :-)"
}

function log()
{
    if [ ! -f "$LOGFILE" ] ; then
        touch "$LOGFILE"
    fi

    echo "$2 ($3) successfully deploy branch: $1" >> "$LOGFILE"
}

while read oldrev newrev ref
do
    branch=`echo $ref | cut -d/ -f3`

    if [ "$branch" ] ; then
        notify "$branch"

        cd ..
        env -i git checkout $branch &> /dev/null
        env -i git reset --hard &> /dev/null

        author_name=`env -i git log -1 --format=format:%an HEAD`
        author_email=`env -i git log -1 --format=format:%ae HEAD`

        log "$branch" "$author_name" "$author_email"
    fi
done
```

Make it executable:

    chmod +x .git/hooks/post-receive

This hook is all you need to perform steps at "deploy time".

Let's test it!

    cd ../../local-server/app
    git push production master

Enjoy:

    ls -l /absolute/path/to/remote-server/www
    tail /tmp/deploy-app.log

**Note:** [git-deploy](https://github.com/mislav/git-deploy) (almost) does the
same thing, but probably better.

### More on this hook

One may want to send emails to the person who just deployed. That is why
`author_name` and `author_email` are retrieved.

Let's try to change the author's email:

    echo "<h1>Hello, World</h1>" > index.html
    git add !$
    GIT_AUTHOR_EMAIL=foo@example.org git commit -m "Surround title with html tags"
    git push production master

### Deploying to another branch

We can deploy another branch:

    git checkout -b feat-content
    echo "<p>content</p>" >> index.html
    git add !$
    git commit -m "add content"
    git push production feat

Result:

    tail /tmp/deploy-app.log
    cat /absolute/path/to/remote-server/www/index.html


## 2. Deploy With Heroku

We are going to deploy a [Node.JS](http://nodejs.org/) application, that is a
[Chat Example](https://github.com/hunterloftis/chat-example).

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/hunterloftis/chat-example)

### Heroku Toolbelt

Install the [Heroku Toolbelt](https://toolbelt.heroku.com/).

Then:

    heroku login
    heroku git:clone -a <heroku-app-id>
    cd !$

Make changes, then deploy:

    git add .
    git commit -m "Make it better"
    git push heroku master


## 3. Deploy With Dokku

[Dokku](https://github.com/progrium/dokku), Docker powered mini-Heroku in around
100 lines of Bash.

Clone the Chat Example:

    git clone git://github.com/hunterloftis/chat-example.git

Deploy it with Dokku.

## 4. Capistrano

...
