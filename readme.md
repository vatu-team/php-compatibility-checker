# PHP Compatibility Checker

Action for checking projects for compatibility with different PHP versions.

## Features

- PHP Linting
- PHP Compatibility

## Install

Add a GitHub Action workflow file to your project (e.g. `./.github/workflows/php-compatibility.yml`):

```yaml
# yaml-language-server: $schema=https://json.schemastore.org/github-workflow.json

on:
  workflow_dispatch:
    inputs:

      runs-on:
        description: "Runner to be used by the parent workflow."
        required: false
        type: string
        default: 'ubuntu-latest'

      branch:
        type: string
        description: "Choose the branch to run this test against."
        default: 'main'

      php-version:
        description: "Version of PHP to test against. Eg, `['8.2','8.3']`"
        required: false
        type: string
        default: "['8.0','8.1','8.2','8.3']"

concurrency:
  group: ${{ github.workflow }}--${{ github.head_ref || github.ref_name }}
  cancel-in-progress: true

jobs:

  php-compatibility:
    runs-on: ${{ inputs.runs-on }}

    strategy:
      fail-fast: true
      max-parallel: 1
      matrix:
        php-version: ${{ fromJson(inputs.php-version) }}

    name: "PHP ${{ matrix.php-version }}"
    timeout-minutes: 15

    steps:

      - name: "Checks out the repository."
        uses: "actions/checkout@v3.5.0"
        with:
          ref: ${{ inputs.branch }}

      - name: "Set up PHP ${{ matrix.php-version }}"
        uses: 'shivammathur/setup-php@2.24.0'
        with:
          php-version: ${{ matrix.php-version }}
          coverage: none
          tools: composer
          ini-file: none
          ini-values: memory_limit=-1

      - name: "Build site."
        run: composer update --no-interaction --prefer-dist --no-scripts --no-dev --ignore-platform-req=php

      - name: 'Initialize Composer.'
        run: |
          mkdir -p tools/php-compat
          composer --working-dir=tools/php-compat init --name "vatu/php-compatibility-checker" --stability stable

      - name: 'Run PHP Lint Tests.'
        uses: 'vatu-team/php-compatibility-checker/php-lint@0.1.0'
        with:
          php-version: ${{ matrix.php-version }}

      - name: 'Run PHP Compatibility Tests.'
        uses: 'vatu-team/php-compatibility-checker/php-compatibility@0.1.0'
        with:
          php-version: ${{ matrix.php-version }}
```

## Versioning

Follows [Semantic Versioning](https://semver.org/). `trunk` is the equivalent
of a nightly release. While stable, should not be used on live projects.

Each workflow has an individual version in the comments of the file header.
These correspond to the release it was last updated in.

## Reporting Issues

If you spot any issues please create a ticket via the project's Issue Tracker.
Including the issue, the browser and operating system, and how to replicate it.
If the issue is security related please use the contact information below.

## License

This package is released under the [MIT license](LICENSE).

## Contact

Vatu - [hello@vatu.dev](hello@vatu.dev)

## Copyright

Copyright (c) 2023 Vatu Limited.
