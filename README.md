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

jobs:
  ci:
    uses: silverstripe/gha-ci/.github/workflows/ci.yml@v1
```

Set config specific to your needs via "inputs" defined under the `with:` key. For instance, to disable PHP linting because your module does not yet have a `phpcs.xml.dist` file

```yml
jobs:
  ci:
    uses: silverstripe/gha-ci/.github/workflows/ci.yml@v1
    with:
      phplinting: false
```

#### Inputs:

##### Extra composer requirements
Require additional modules through composer for matrix entries. You do not need to quote the string for multiple requirements
`composer_require_extra: silverstripe/widgets:^2 silverstripe/comments:^3`

##### Simple matrix
Create a smaller matrix with only the lowest supported PHP and MySQL versions instead of a full matrix of multiple PHP and database versions. Default is false, enable with:
`simple_matrix: true`

##### Dynamic matrix
Dynamically generate a matrix using feature detection. If disabled, jobs must be defined using the extra_jobs input. Default is true, disable with:
`dynamic_matrix: false`

##### PHPUnit tests
Runs PHPunit if the `phpunit.xml` or `phpunit.xml.dist` config file is available. Default is true, disable PHPunit tests with:
`phpunit: false`

##### PHP linting
Runs phpcs and phpstan if the `phpcs.xml.dist` or `phpstan.neon.dist` config files are available. Default is true, disable with:
`phplinting: false`

##### PHP coverage
Run codecov, which does not require a configuration file. Default is false, though modules on the silverstripe account will always have this enabled. Enable with:
`phpcoverage: true`

##### End-to-end tests
Runs behat tests if `behat.yml` is available. Default is true, disable with:
`endtoend: false`

##### JS tests
Runs `yarn lint`, `yarn test` and `yarn build` + `git diff-files` + `git diff` if the package.json file is available. Default is true, disable JS tests with:
`js: false`

##### Extra jobs
Define specific test combinations (e.g. php and db versions). All inputs are false by default so you only need to include config for the input(s) you want enabled. All inputs except `dynamic_matrix` and `simple_matrix` are available in this context. There are also some additional inputs specific to the extra_jobs input (see below). Use a multi-line yml string
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

For the list of supported databases see [the gha-generate-matrix script](https://github.com/silverstripe/gha-generate-matrix/blob/main/job_creator.php)

###### name_suffix
Add a specific suffix to job and artifact names to make them easier to identify. This will be trucated if greater than 20 characters. Must match the regex `[a-zA-Z0-9_\- ]`

###### phpunit_suite
The PHPUnit suite to run as defined in `phpunit.xml`. Translates to the `--testsuite` parameter passed to `phpunit`. Specify 'all' to run all PHPunit suites available in the context of this module/recipe

###### endtoend_suite
The end-to-end suite as defined in `behat.yml`. Must be defined, specify 'root' to run the default suite

###### endtoend_config
The `behat.yml` config file to use if not using the root `behat.yml`. Only used if running behat tests in a different module
