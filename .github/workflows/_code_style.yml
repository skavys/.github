name: Code Style

# TODO: make auto-commit for check-style
# TODO: check all caches

on:
  workflow_call:
    inputs:
      cache_key:
        description: "PHP extensions cache key"
        required: true
        type: string
      extensions:
        description: "PHP extensions"
        default: simplexml, mbstring
        required: false
        type: string
      tools:
        description: "PHP tools"
        required: false
        default: composer, cs2pr, laravel/pint
        type: string
      ini_values:
        description: "PHP ini values"
        default: max_execution_time=180, date.timezone=Asia/Tbilisi
        required: false
        type: string

jobs:
  code-style:
    timeout-minutes: 15
    if: github.event.pull_request.draft == false
    name: code-style
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ ubuntu-latest ]
        php-version: [ 8.1 ]

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup cache environment
        id: ext-cache
        uses: shivammathur/cache-extensions@v1
        with:
          php-version: ${{ matrix.php-version }}
          extensions: ${{ inputs.extensions }}
          key: ${{ inputs.cache_key }}

      - name: Cache extensions
        uses: actions/cache@v3
        with:
          path: ${{ steps.ext-cache.outputs.dir }}
          key: ${{ steps.ext-cache.outputs.key }}
          restore-keys: ${{ steps.ext-cache.outputs.key }}

      - name: Setup PHP version ${{ matrix.php-version }} on ${{ matrix.os }}
        id: setup-php
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}
          extensions: ${{ inputs.extensions }}
          tools: ${{ inputs.tools }}
          coverage: none
          ini-values: ${{ inputs.ini_values }}
        env:
          COMPOSER_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          update: true

      - name: Setup problem matchers
        run: echo "::add-matcher::${{ runner.tool_cache }}/php.json"

      - name: Validate composer.json and composer.lock
        run: composer validate --strict --ansi

      - name: Composer Install
        id: composer-install
        uses: ramsey/composer-install@v2
        with:
          dependency-versions: "locked"
          composer-options: "--prefer-dist"

      - name: Check PHP code style
        id: cs-check
        run: pint --test -v

      - name: Generate Annotations on CS errors
        if: failure() && steps.cs-check.outcome != 'success'
        run: pint --test --format=checkstyle | cs2pr
