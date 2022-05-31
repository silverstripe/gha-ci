# GitHub Actions - CI

CI used by Silverstripe modules

Will use feature detection based on files in the root folder such as phpunit.xml.dist to build dynmaic matrix of tests to run

It's highly recommended that you use a tagged version (e.g. v0.2) to ensure stability of your builds. If you have a relatively simple build that you have no intention of ever making more complex e.g. only phpunit tests using phpunit.xml.dist, then this is probably all you need for long term use.

This repository is currently in development and code on the `main` branch could change at any time, including taking on a whole new direction. It's expected that new functionality will be added.

Note: Unlike other silverstripe/gha-* repositiories, this one is a [reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows) rather than an [action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action). A reusable workflow is required to create a `matrix`.

### CI Usage

Create the following file in your module

(subsitute the tagged version for the most recent tag from this module)

**.github/workflows/ci.yml**
```yml
name: CI

on:
  push:
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    uses: silverstripe/ghi-ci/.github/workflows/ci.yml@main
```

Use the following if your module does not have a `phpcs.xml.dist` file

(or better still, copy paste this [sample phpcs.xml.dist](https://raw.githubusercontent.com/silverstripe/silverstripe-elemental/4/phpcs.xml.dist) file in to your module)

```yml
jobs:
  ci:
    uses: silverstripe/github-actions-ci-cd/.github/workflows/ci.yml@v0.2
    with:
      phplinting: false
```

#### Some other "with" options

Extra composer requirements
You do not need to quote the string for multiple requirements
`composer_require_extra: silverstripe/widgets:^2 silverstripe/comments:^3`

Simple matrix - php 7.4 with mysql 5.7 only
`simple_matrix: true`

Enable php coverage (codecov - no feature detection)
Modules on the silverstripe account will automaticaly have this enabled
`phpcoverage: true`

Disable end-to-end tests (behat.yml):
`endtoend: false`

Disable JS tests (package.json - yarn lint, test and build diff):
`js: false`

Disable phpunit tests (phpunit.xml.dist / phpunit.xml)
`phpunit: false`

Disable php linting (phpcs.xml.dist, phpstan.neon.dist)
`phplinting: false`

Extra jobs
Define php version and/or db
```yml
extra_jobs: |
  - php: '8.0'
    endtoend: true
```
