# GitHub Actions - CI

CI for Silverstripe modules

Uses feature detection based on files in the root folder such as phpunit.xml.dist to build and run a dynamic matrix of jobs.

It's highly recommended that you use a tagged version (e.g. v1) to ensure stability of your builds. If you have a relatively simple build that you have no intention of ever making more complex e.g. only phpunit tests using phpunit.xml.dist, then this is probably all you need for long term use.

Note: Unlike other silverstripe/gha-* repositories, this one is a [reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows) rather than an [action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action). A reusable workflow is required to create a `matrix`.

### Usage

Create the following file in your module, and substitute the tagged version for the most recent tag prefixed with a `v` e.g. `@v1`

**.github/workflows/ci.yml**
```yml
name: CI

on:
  push:
  pull_request:
  workflow_dispatch:

permissions: {}

jobs:
  ci:
    name: CI
    permissions:
      pull-requests: read
      contents: read
    uses: silverstripe/gha-ci/.github/workflows/ci.yml@v1
```

#### Running on a regular schedule

```yml
on:
  # Run once per week
  schedule:
  - cron: '0 0 * * 1'

permissions: {}

jobs:
  ci:
    name: CI
    permissions:
      pull-requests: read
      contents: read
    # Only run the cron on the account hosting this repository, not on the accounts of forks
    # Change '<account_name>' to match the name of the account hosting this repository
    if: (github.event_name == 'schedule' && github.repository_owner == '<account_name>') || (github.event_name != 'schedule')
    uses: silverstripe/gha-ci/.github/workflows/ci.yml@v1
```

#### Job configuration

Set config specific to your needs via "inputs" defined under the `with:` key. For instance, to disable PHP linting because your module does not yet have a `phpcs.xml.dist` file

```yml
jobs:
  ci:
    name: CI
    uses: silverstripe/gha-ci/.github/workflows/ci.yml@v1
    with:
      phplinting: false
```

#### Inputs:

##### Extra composer requirements
Require additional modules through composer for matrix entries. You do not need to quote the string for multiple requirements
`composer_require_extra: silverstripe/widgets:^2 silverstripe/comments:^3`

##### Composer install
Use `composer install` rather than default `composer update`. A composer.lock file must be present. Intended only for testing branches of `silverstripe/installer` or `silverstripe/recipe-kitchen-sink` where multiple pull-requests need to be tested together. If `true` will cause `composer_require_extra` input to be ignored. If `dynamic_matrix` input also is `true` then jobs will only be created for the PHP version defined in `config.platform.php` in composer.json (recommended), or the lowest supported PHP version. There will also be no `--prefer-lowest` jobs created. Default is false, enable with:
`composer_install: true`

##### Simple matrix
Create a smaller matrix with only the lowest supported PHP and MySQL versions instead of a full matrix of multiple PHP and database versions. Default is false, enable with:
`simple_matrix: true`

##### Dynamic matrix
Dynamically generate a matrix using feature detection. If disabled, jobs must be defined using the extra_jobs input. Default is true, disable with:
`dynamic_matrix: false`

##### Documentation linting
Runs linting against documentation using [silverstripe/documentation-lint](https://github.com/silverstripe/documentation-lint/) if the `.doclintrc` config file is available. Default is true, disable with:
`doclinting: false`

##### PHPUnit tests
Runs PHPunit if the `phpunit.xml` or `phpunit.xml.dist` config file is available. Default is true, disable PHPunit tests with:
`phpunit: false`

##### Skip PHPUnit suites
Skips specific PHPUnit suites in the dynamically generated matrix. Default is to run all suites. Configure with:
`phpunit_skip_suites: suite-to-skip,another-to-skip`

##### PHP linting
Runs phpcs and phpstan if the `phpcs.xml.dist` or `phpstan.neon.dist` config files are available. Default is true, disable with:
`phplinting: false`

##### PHP coverage
Run codecov, which does not require a configuration file. Default is false, though for CI runs on the silverstripe account this option will be ignored and codecov is set to run in [gha-generate-matrix](https://github.com/silverstripe/gha-generate-matrix). Enable with:
`phpcoverage: true`

##### PHP coverage force off
Force codecov off for CI runs on the silverstripe account. Default is false. Enable with:
`phpcoverage_force_off: true`

##### End-to-end tests
Runs behat tests if `behat.yml` is available. Default is true, disable with:
`endtoend: false`

##### JS tests
Runs `yarn lint`, `yarn test` and `yarn build` + `git diff-files` + `git diff` if the package.json file is available. Default is true, disable JS tests with:
`js: false`

##### Extra jobs
Define specific test combinations (e.g. php and db versions). All inputs are false by default so you only need to include config for the input(s) you want enabled. All inputs except `dynamic_matrix`, `simple_matrix`, and `phpunit_skip_suites` are available in this context. There are also some additional inputs specific to the extra_jobs input (see below). Use a multi-line yml string
```yml
extra_jobs: |
  - php: '8.0'
    db: pgsql
    phpunit: true
    phpunit_suite: api
  - php: '8.1'
    endtoend: true
    endtoend_suite: admin
    endtoend_config: vendor/silverstripe/admin/behat.yml
```

##### Extra jobs - additional inputs
Additional inputs that can be applied to individual jobs within the extra_jobs input

###### php
The version of PHP to run e.g. 8.1

###### db
The database to run e.g. mysql80

Valid `db` are:
- pgsql
- mysql57
- mysql80
- sqlite3
- mariadb

###### name_suffix
Add a specific suffix to job and artifact names to make them easier to identify. This will be trucated if greater than 20 characters. Must match the regex `[a-zA-Z0-9_\- ]`

###### phpunit_suite
The PHPUnit suite to run as defined in `phpunit.xml`. Translates to the `--testsuite` parameter passed to `phpunit`. Specify 'all' to run all PHPunit suites available in the context of this module/recipe

###### preserve_vendor_tests
By default, the workflow will want try to delete tests for modules in the `vendor` folder to avoid potential conflicts with tests rewritten for different PHPUnit versions. This behaviour is not always desirable because you may want to run those tests for your current build.

Setting this input to `true` will preserve tests in the vendor folder.

###### endtoend_suite
The end-to-end suite as defined in `behat.yml`. Must be defined, specify 'root' to run the default suite

###### endtoend_config
The `behat.yml` config file to use if not using the root `behat.yml`. Only used if running behat tests in a different module

###### install_in_memory_cache_exts
Must be true or false. Determines whether to install in-memory cache extensions for framework unit tests.
