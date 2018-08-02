#!/bin/sh

# setup

## install https://github.com/github/hub ##
```
brew install hub
```
## make it use https vs ssh
```
git config --global hub.protocol https
```
## clone and fork https://github.com/dougbeal/wordpress-indieweb-plugins.git ##

ORG="" # set org here
git clone --recurse-submodules https://github.com/dougbeal/wordpress-indieweb-plugins.git

set -x
set -uo pipefail
repos=(  # fake associative array for bash3
    "wordpress-indieweb-plugins:.:dougbeal"
    "indieweb-post-kinds:indieweb-post-kinds:dshanske"
    "simple-location:simple-location:dshanske"
    "syndication-links:syndication-links:dshanske"
    "wordpress-indieauth:wordpress-indieauth:indieweb"
    "wordpress-indieweb:wordpress-indieweb:indieweb"
    "wordpress-indieweb-press-this:wordpress-indieweb-press-this:indieweb"
    "wordpress-micropub:wordpress-micropub:snarfed"
    "wordpress-semantic-linkbacks:wordpress-semantic-linkbacks:pfefferle"
    "wordpress-webmention:wordpress-webmention:pfefferle"
)


for idx in "${!repos[@]}"; do
    str="${repos[$idx]}"
    upstream_repo=${str%%:*}
    repo_dir=${str%:*}
    repo_dir=${repo_dir##*:}
    upstream_username=${str##*:}
    echo "$idx ${upstream_repo} dir ${repo_dir} user ${upstream_username}"
    (
        cd ${repo_dir}
        git remote -v | grep -q "upstream"
        if [ $? -ne 0 ]; then
            git remote add upstream https://github.com/${upstream_username}/${upstream_repo}.git 
            #git remote set-url upstream https://github.com/${upstream_username}/${upstream_repo}.git
        fi
        
    )
    if [ "${repo_dir}" != "." ]; then
        git config --file=.gitmodules submodule.${upstream_repo}.url | grep -q "${upstream_username}"
        if [ $? -ne 0 ]; then
            echo "fixing .gitmodule submoduel url for ${upstream_repo}"
            git config --file=.gitmodules submodule.${upstream_repo}.url https://github.com/${upstream_username}/${upstream_repo}.git
        fi
    fi        
done

hub fork ${ORG}

## create working branch (your github username?) ##
GITHUB_USER=${GITHUB_USER:-$USER}
GIT_BRANCHNAME=${USER:-"my-working-branch"}
git checkout -b ${GIT_BRANCHNAME} || git checkout ${GIT_BRANCHNAME} # checkout and create, or checkout if it already exists

## fork each indieweb plugin to your account ##

git submodule foreach hub fork ${ORG}

git submodule foreach git checkout -b ${GIT_BRANCHNAME}
git submodule foreach git push --set-upstream origin ${GIT_BRANCHNAME}

## update submodule to point to user's fork, and branch to user's new branchrepo to point to users and new branch
for submodule in $(git submodule | gawk '{print $2}'); do
    echo "update ${submodule} for user ${GITHUB_USER} and branch ${GIT_BRANCHNAME}"
    git config --file=.gitmodules \
        submodule.${submodule}.url \
        https://github.com/${GITHUB_USER}/${submodule}.git
    git config --file=.gitmodules submodule.${submodule}.branch ${GIT_BRANCHNAME}
done

## push branch to your your repo ##

git push --set-upstream origin ${GIT_BRANCHNAME}

# update your branch to upstream

## update orign
git submodule foreach hub sync

## merge oring into ${GIT_BRANCHNAME}
git submodule foreach git merge origin/master

git submodule foreach git push -u ${GIT_BRANCHNAME} ${GIT_BRANCHNAME}


# run tests
docker exec iwwp-plugins_wordpress_1 bash -c "find \${WORDPRESS_PATH} -maxdepth 5 -name phpunit.xml\* -print -exec  phpunit  --config {} \;" > test.report


# for wordpress-indieweb-plugins/master, point submodules at upstream/master
use name upsream-master to avoid conflict with origin/master
```
# will fail if upstream-master already exists, so do || true
git submodule foreach bash -c "git fetch upstream && git checkout -b upstream-master upstream/master || true"

# verify that the repos are set correctly
git submodule foreach git branch -v
```

# helpful config 
```
#  show the submodule log when you dif
git config --global diff.submodule log

# short summary of submodule changes in your status message:
git config status.submoduleSummary true
```
