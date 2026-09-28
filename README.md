# drupal-module-ci

A reusable GitHub Actions workflow for Drupal modules and themes that live in
their own repository. It builds a fresh Drupal site with Composer, copies your
code into it, and runs the same three checks drupal.org's GitLab CI runs:

- **phpcs** with the `Drupal` and `DrupalPractice` standards
- **phpstan** with `phpstan-drupal` and the deprecation rules
- **PHPUnit** (Unit, Kernel and Functional; a PHP built-in server is started
  for Functional tests)

across a matrix of PHP and Drupal versions. Defaults today: PHP 8.3 and 8.4
against Drupal `^10.6` and `^11.4`.

The caller repo needs no `composer.lock`, no vendor directory and no Drupal
core checked in.

## Use it

`.github/workflows/ci.yml` in the module repo:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  drupal:
    uses: joehahndev/drupal-module-ci/.github/workflows/drupal.yml@v1
    with:
      install-map: |
        .:web/modules/custom/my_module
      phpstan-level: '6'
```

A repo that holds both a module and a theme maps each one:

```yaml
    with:
      install-map: |
        modules/drupress:web/modules/custom/drupress
        themes/drupress_admin:web/themes/custom/drupress_admin
```

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `install-map` | whole repo → `web/modules/custom/<repo name>` | `source:destination` lines. Source is relative to the repo, destination to the built site. |
| `php-versions` | `["8.3", "8.4"]` | JSON array. |
| `drupal-versions` | `["^10.6", "^11.4"]` | JSON array of Composer constraints for `drupal/core`. |
| `phpstan-level` | `6` | PHPStan rule level. Start lower on inherited code. |
| `phpstan-ignore` | empty | Extra `ignoreErrors` lines. `identifier: foo.bar` or a message regex. |
| `phpcs-extensions` | `php,module,inc,install,test,profile,theme,css,info,txt,md,yml` | What phpcs scans. |
| `phpunit` | `true` | Run PHPUnit when a `tests/` directory exists in any installed path. |
| `composer-require` | empty | Extra packages the module depends on, space separated, e.g. `drupal/token drupal/pathauto`. |

## Why a fresh site every run

Static analysis of Drupal code needs Drupal core on disk: `phpstan-drupal`
resolves services, entity types and hooks from it, and PHPUnit's Drupal test
base classes bootstrap a real (SQLite) site. Building the site inside the
workflow keeps the module repo small and means the matrix tests real core
versions, not whatever one developer happened to have installed.

## Pinning

`@v1` tracks the latest 1.x. Pin `@v1.0.0` if you want nothing to move.

## License

MIT.
