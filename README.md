<a href="https://newfold.com/" target="_blank">
    <img src="https://newfold.com/content/experience-fragments/newfold/site-header/master/_jcr_content/root/header/logo.coreimg.svg/1621395071423/newfold-digital.svg" alt="Newfold Logo" title="Newfold Digital" align="right" 
height="42" />
</a>

# Newfold Satis Repository

[Satis](https://composer.github.io/satis/) is an open-source [Composer](https://getcomposer.org/) repository generator. This repository serves as Newfold's static file-based version of [Packagist](https://packagist.org/) and hosts the metadata of Newfold's packages.

## Usage

- Run `composer config repositories.newfold-labs composer https://newfold-labs.github.io/satis` in your project to access available packages.
- Head over to https://newfold-labs.github.io/satis/ to browse available packages.
- Run `composer require newfold-labs/{packageName}` to install a package.

## Adding A Repo

- Clone this repository: `git clone git@github.com:newfold-labs/satis.git`
- Run `composer install`
- Run `composer satis add <url>` where `<url>` is the URL for the Git repo. SSH GitHub URLs are preferred.
- Commit changes and push.
- In the remote repo, setup the following GitHub Action so that new releases will trigger Satis to rebuild.

```
name: Trigger Satis Build

on:
  release:
    types:
      - created

jobs:
  webhook:
    name: Send Webhook
    runs-on: ubuntu-latest
    steps:

    - name: Set Package
      id: package
      env:
        REPO: ${{ github.repository }}
      run: echo "PACKAGE=${REPO##*/}" >> $GITHUB_OUTPUT

    - name: Set Version
      id: tag
      run: echo "VERSION=${GITHUB_REF##*/}" >> $GITHUB_OUTPUT

    - name: Repository Dispatch
      uses: peter-evans/repository-dispatch@v1
      with:
        token: ${{ secrets.WEBHOOK_TOKEN }}
        repository: newfold-labs/satis
        event-type: 'satis-build'
        client-payload: >-
          {
            "vendor": "${{ github.repository_owner }}",
            "package": "${{ steps.package.outputs.PACKAGE }}",
            "version": "${{ steps.tag.outputs.VERSION }}"
          }

```

You must create a [personal access token](https://github.com/settings/tokens) with `repo` access and set it as the `WEBHOOK_TOKEN` secret on the remote repository. Also, make sure that GitHub Actions is enabled for the repository.