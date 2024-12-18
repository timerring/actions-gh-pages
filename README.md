<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [
GitHub Pages Action with GPG verify
](#github-pages-action-with-gpg-verify)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

<h2 align="center">
GitHub Pages Action with GPG verify
</h2>

This is a adjusted version of [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages), you can refer to the original repository documentation.

Here I will describe my yml file to deploy it with GPG verify, combining with [crazy-max/ghaction-import-gpg](https://github.com/crazy-max/ghaction-import-gpg).


```yaml
name: deploy

on:
    push:
        branches:
            - main
    workflow_dispatch:

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout
              uses: actions/checkout@v3
              with:
                  submodules: true
                  fetch-depth: 0
                  ref: main

            - name: Setup Hugo
              uses: peaceiris/actions-hugo@v2
              with:
                  hugo-version: "0.108.0"
                  extended: true

            - name: Build Web
              run: hugo --minify

            - name: Import GPG key # import the gpg key to the github action
              uses: crazy-max/ghaction-import-gpg@v6 # repository https://github.com/crazy-max/ghaction-import-gpg
              with: # I use the subkey to sign the commit, if you use the primary key, you can refer to his repository docs.
                  gpg_private_key: ${{ secrets.GPG_PRIVATE_KEY }} # the secret gpg subkey
                  passphrase: ${{ secrets.PASSPHRASE }} # the passphrase of the gpg subkey
                  git_user_signingkey: true
                  git_commit_gpgsign: true
                  fingerprint: ${{ secrets.FINGERPRINT }} # the fingerprint of the public subkey you use

            - name: Deploy Web
              uses: timerring/actions-gh-pages@v5.0.0 # this is adjusted action from peaceiris/actions-gh-pages, you can use it directly.
              with:
                  personal_token: ${{ secrets.PERSONAL_TOKEN }} # the personal token of the github action
                  external_repository: your_username/your_repository # your target repository
                  publish_branch: main # the branch you want to deploy
                  publish_dir: ./public # the directory you want to deploy
                  user_name: ${{ secrets.USER_NAME }} # the name of the github action
                  user_email: ${{ secrets.USER_EMAIL }} # the email of the github action # ATTENTION: please add your github verified email
                  commit_message: ${{ github.event.head_commit.message }}
```

> [!WARNING]
> Don't forget to add the corresponding `secrets.XXX` variables in the repository settings.


For more information, you can refer to my blog: [deploy-github-pages-with-gpg-signing](https://timerring.github.io/blog/posts/2024/deploy-github-pages-with-gpg-signing).
