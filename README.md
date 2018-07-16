#!/bin/sh

# setup

## install https://github.com/github/hub ##

## make it use https vs ssh
git config --global hub.protocol https

## clone and fork https://github.com/dougbeal/wordpress-indieweb-plugins.git ##

ORG="" # set org here
git clone https://github.com/dougbeal/wordpress-indieweb-plugins.git
hub fork ${ORG}

## create working branch (your github username?) ##

BRANCHNAME=${USER:-"my-working-branch"}
git checkout -b ${BRANCHNAME} || git checkout ${BRANCHNAME}

## fork each indieweb plugin to your account ##

git submodule foreach hub fork ${ORG}

git submodule foreach git checkout -b ${BRANCHNAME}
git submodule foreach git push --set-upstream origin ${BRANCHNAME}

## push branch to your your repo ##

git push --set-upstream origin ${BRANCHNAME}

# update your branch to upstream

## update orign
git submodule foreach hub sync

## merge oring into ${BRANCHNAME}
git submodule foreach git merge origin/master

git submodule foreach git push -u ${BRANCHNAME} ${BRANCHNAME}
