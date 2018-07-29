#!/bin/sh

# setup

## install https://github.com/github/hub ##

## make it use https vs ssh
git config --global hub.protocol https

## clone and fork https://github.com/dougbeal/wordpress-indieweb-plugins.git ##

ORG="" # set org here
git clone --recursive https://github.com/dougbeal/wordpress-indieweb-plugins.git

# upstream repos (upstream: canonical, origin: your repo, USERNAME: your repo) [no associative in bash 3]
declare -a upstream_repos=(
    wordpress-indieweb-plugins
    indieweb-post-kinds
    simple-location
    syndication-links
    wordpress-indieweb-press-this
    wordpress-indieweb
    wordpress-semantic-linkbacks
    wordpress-webmention
    wordpress-micropub
)

declare -a repo_directory=(
    .
    indieweb-post-kinds
    simple-location
    syndication-links
    wordpress-indieweb-press-this
    wordpress-indieweb
    wordpress-semantic-linkbacks
    wordpress-webmention
    wordpress-micropub
)

declare -a upstream_usernames=(
    dougbeal
    dshanske
    dshanske
    dshanske
    indieweb
    indieweb
    pfefferle
    pfefferle
    snarfed
)


for idx in "${!upstream_repos[@]}"; do
    echo "$idx"
    echo "repo ${upstream_repos[$idx]}"
    echo "repo ${repo_directory[$idx]}"    
    echo "user ${upstream_usernames[$idx]}"
    (
        cd ${repo_directory[$idx]}
        git remote add upstream https://github.com/${upstream_usernames[$idx]}/${upstream_repos[$idx]}.git
    )
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
