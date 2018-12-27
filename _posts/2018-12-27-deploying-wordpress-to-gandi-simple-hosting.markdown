---
layout: post
categories: git
title: Deploying WordPress to Gandi Simple Hosting with GitLab CI
tags: [gitlab, wordpress]
---
![WP-Gandi-Gitlab](/img/wp-gandi-gitlab.png)
For this experiment I wanted to automate a Wordpress website installation using GIT into my Gandi Simple Hosting instance. Along the way I came across several roadblocks which led me to using GitLab CI/CD which I had been wanting to try for some time so I thought this was a good exercise for my curiosity and below I describe how I got it working.<!--more-->

# Setting WordPress as a GIT Submodule
In order to reduce my repository size to a minimum, one of the things I wanted to achieve was to use WordPress itself as a GIT submodule in a similar way as what I read in [this article](https://deliciousbrains.com/install-wordpress-subdirectory-composer-git-submodule/){:target="_blank"}. One of the modifications I made in my implementation was to have an `htdocs` directory in the root of my repository to comply with Gandi's hosting file structure. Running Gandi's deploy instructions I ran into my first roadblock: Gandi does not yet support submodules in their GIT deployments. This is a bummer but not a huge deal since I wanted to try out GitLab CD anyway.

# Configuring GitLab Continuous Delivery
After pushing my initial repository to Gitlab, one of the first things I had to do is to give GitLab permissions to upload files to Gandi which I did using an SSH key. For this I generated a key pair with the following command:
```
$ ssh-keygen -t rsa
```
When prompted, enter the path to the file where to store the generated keys and then skip the passphrase prompts hitting enter. A private key file and public key file (with `.pub`) will be generated. You should then upload the public ssh key to your Gandi keychain and then store the private key in a GitLab variable called `SSH_PRIVATE_KEY` in _Settings > CI/CD_ in your repository page.  Note, when copying he private key into a GitLab variable, make sure to copy the entire contents of the key file including the `-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----` parts otherwise the job will fail, there are some comments about this [here](https://gitlab.com/gitlab-examples/ssh-private-key/issues/1#note_20786515){:target="_blank"}.

Next, we will create a `.gitlab-ci.yml` file in our repository root and add the instructions make use of our SSH key as described [here](https://docs.gitlab.com/ee/ci/ssh_keys/){:target="_blank"}. The `.gitlab-ci.yml` file should look similar to this (removing comment lines):
```
variables:
    GIT_SUBMODULE_STRATEGY: recursive

before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -> /dev/null
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
```
The above commands will be run before any other runner is executed and they take care of installing `ssh-agent` (if not already installed in the runner) and adding the SSH private key to be able to send files to Gandi. In addition to this, I have set `GIT_SUBMODULE_STRATEGY` to recursive so that the submodules are also downloaded before sending code later.

# Adding deploy stage to upload files
```
deploy:
    stage: deploy
    only:
        - master
    script:
        - scp -o stricthostkeychecking=no -r . $REMOTE_SCP_ADDRESS
```
We need to now tell the runner what to do and when, so the above will automatically create a "deploy" (you can call it anything) stage that will run the script within the `script` tag when changes are pushed to the `master` branch. I also added `-o stricthostkeychecking=no` to avoid getting errors due to host key checks and `-r` for recursive copy of directories into the `REMOTE_SCP_ADDRESS`. I used a Gitlab variable for the remote address to be able to edit it from the Gitlab repository settings which should be similar to this: `12345@sftp.sd6.gpaas.net:/lamp0/web/vhosts/www.domain.com`. Further on, I split this variable into `REMOTE_USER`, `REMOTE_HOST` and `REMOTE_BASEDIR`.

I tried the above but I then got the following error:
```
$ scp -o stricthostkeychecking=no -r . $REMOTE_SCP_ADDRESS
Warning: Permanently added 'sftp.sd6.gpaas.net,155.133.142.129' (ECDSA) to the list of known hosts.
Permission denied, please try again.
Permission denied, please try again.
Permission denied (publickey,password).
lost connection
ERROR: Job failed: exit code 1
```

Gandi not only doesn't support GIT submodules but it doesn't support SCP either, it **only** supports SFTP. The above would therefore work for other servers and so should `rsync` too but not with Gandi so we need an alternative via sftp.

# Setting up a recursive SFTP upload job
SFTP can be quite handy to upload files and I grabbed some hints from [this post](https://unix.stackexchange.com/questions/55190/how-to-send-multiple-commands-to-sftp-using-one-line){:target:"_blank"} to come up with the following one-liner command:
```
echo -e "cd $REMOTE_BASEDIR\nput *" | sftp -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST
```
One of the drawbacks of SFTP though is that it won't automatically create missing directories when doing a recursive upload so doing a little research I came across LFTP which is a program that supports SFTP and can handle recursive upload. On [this](https://unix.stackexchange.com/questions/181781/using-lftp-with-ssh-agent){:target="_blank"} post I also found some more info on how to make it work with my SSH key and with info from [here](https://stackoverflow.com/questions/1595565/how-to-implement-recursive-put-in-sftp){:target="_blank"} I completed my code to recursively send files.

```
lftp -c "set sftp:auto-confirm yes; open -u $REMOTE_USER, sftp://$REMOTE_HOST$REMOTE_BASEDIR ; mirror -R -v ./"
```
In the above `set sftp:auto-confirm yes` takes care of skipping host key checks, `mirror` is the command to copy recursively, `-R` (reverse mirrir) to upload instead of download and you can add `-v` to get verbose output in the log of what is being sent. Once we make sure the command works as expected, we can add `-e` to delete files in the destination which are not in the source, but I first skipped it to make sure I didn't accidentally delete other directories. Later we can also add `--exclude .git --exclude .gitlab-ci.yml --exclude .gitmodules --exclude .gitignore` to the `mirror` command to avoid sending unwanted files to our hosting.

Note that LFTP might not be installed by default in the runner instance so we must add the following to the `before_script` code:
```
which lftp || ( apt-get update -y && apt-get install lftp -y )
```

Overall, the contents of the `.gitlab-ci.yml` should now be similar to this:
```
variables:
    GIT_SUBMODULE_STRATEGY: recursive

before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - 'which lftp || ( apt-get update -y && apt-get install lftp -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -> /dev/null
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh

deploy:
    stage: deploy
    only:
        - master
    script:
        - lftp -c "set sftp:auto-confirm yes; open -u $REMOTE_USER, sftp://$REMOTE_HOST$REMOTE_BASEDIR ; mirror -Rve --exclude .git --exclude .gitlab-ci.yml --exclude .gitmodules --exclude .gitignore ./"
```

# Optimizing code delivery times
With the above we managed to automate WordPress code deployment to our server but it is far from efficient. Every time you deploy code, the entire WordPress installation is sent over SFTP, this process is far from quick and also takes time from your free GitLab pipelines quota so below I optimized the process by only sending changed files and by creating a separate stage only for the WordPress submodule update:

Only sending changed files is simple, we can add `--ignore-time --only-newer` to the lftp `mirror` command and only files which have different filesize will be sent. I still need to check what happens if some file has changed but the filesize remains the same, but I will leave this for later.

In order to separate our pipelines in two stages we update the `gitlab-ci.yml` file as below:

```
variables:
    GIT_SUBMODULE_STRATEGY: recursive

before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - 'which lftp || ( apt-get update -y && apt-get install lftp -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -> /dev/null
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh

stages:
    - deploy
    - wordpress-update

wordpress-update:
    stage: wordpress-update
    only:
        refs:
            - master
        changes:
            - htdocs/wp
    script:
        - lftp -c "set sftp:auto-confirm yes; open -u $REMOTE_USER, sftp://$REMOTE_HOST$REMOTE_BASEDIR/htdocs/wp ; mirror -Rve --ignore-time --only-newer --exclude wp-content ./htdocs/wp"

deploy:
    stage: deploy
    only:
        refs:
            - master
    script:
        - lftp -c "set sftp:auto-confirm yes; open -u $REMOTE_USER, sftp://$REMOTE_HOST$REMOTE_BASEDIR ; mirror -Rve --ignore-time --only-newer --exclude .git --exclude .gitlab-ci.yml --exclude .gitmodules --exclude .gitignore --exclude htdocs/wp --exclude htdocs/content/uploads ./"

```
In the code above we have created an additional `wordpress-update` stage which will only run when changes happen to `htdocs/wp` (the WP submodule) in the `master` branch. Then in the deploy stage we have added `--exclude htdocs/wp` to skip WP in this stage. Additionally I also excluded the uploads directory, not to overwrite uploaded files in production and also skipped the native wp-content directory from the WP submodule since we are not using that location anymore.

With all the above, everything should be automated nicely and we should be able to test locally plugin and theme updates and then committing to master in order to get them automatically uploaded to our live server. Similarly, when updating WordPress we just need to checkout the required tag in the submodule and push the changes to the master branch; GitLab will take care of the rest.

There are still some considerations or additional stages that we could add to our script, such as creating a backup of the database for every WP version update, placing a "under maintenance" page during the deployment, etc. but I will leave that for another time.

# Resources
- https://deliciousbrains.com/install-wordpress-subdirectory-composer-git-submodule/
- https://docs.gitlab.com/ee/ci/ssh_keys/
- https://gitlab.com/gitlab-examples/ssh-private-key/issues/1#note_48526556
- https://unix.stackexchange.com/questions/55190/how-to-send-multiple-commands-to-sftp-using-one-line
- https://serverfault.com/questions/330503/scp-without-known-hosts-check
- https://unix.stackexchange.com/questions/181781/using-lftp-with-ssh-agent
- https://stackoverflow.com/questions/1595565/how-to-implement-recursive-put-in-sftp
- https://stackoverflow.com/questions/11490145/why-lftp-mirror-only-newer-does-not-transfer-only-newer-file/15320869
