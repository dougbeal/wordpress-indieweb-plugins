#!/bin/sh
# install https://github.com/github/hub

# clone and fork https://github.com/dougbeal/wordpress-indieweb-plugins.git
ORG="" # set org here
git clone https://github.com/dougbeal/wordpress-indieweb-plugins.git
hub fork ${ORG}

# create working branch (your github username?)
BRANCHNAME=${USER:-"my-working-branch"}
git checkout -b ${BRANCHNAME}

# fork each indieweb plugin to your account
git submodule foreach hub fork ${ORG}
