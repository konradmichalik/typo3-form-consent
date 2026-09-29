# AGENTS.md

## Project overview

TYPO3 extension (`form_consent`) that adds double opt-in to EXT:form. A custom `Consent` form finisher stores the submitted form data as JSON, sends a confirmation mail and a plugin approves or dismisses the consent. Events allow adapting the whole process. A scheduler task removes expired consents (GDPR) and a dashboard widget shows consent states.

- Package: `eliashaeussler/typo3-form-consent`, namespace `EliasHaeussler\Typo3FormConsent` (PSR-4, `Classes/`)
- Requirements: PHP 8.2 to 8.5, TYPO3 13.4 and 14.3
- Dependencies, Composer vendor dir and binaries live in `.Build/`

## Structure

- `Classes/Domain/` `Finishers/` (Consent finisher), `Model/`, `Repository/`, `Factory/`
- `Classes/Controller/` `ConsentController` (approve and dismiss plugin)
- `Classes/Event/` PSR-14 events (approve, dismiss, modify consent, modify mail, generate hash)
- `Classes/Service/` hash service, `Classes/Registry/`, `Classes/Type/` (DBAL types), `Classes/Enums/`
- `Classes/Form/`, `Classes/Widget/`, `Classes/ExpressionLanguage/`, `Classes/Compatibility/`, `Classes/DependencyInjection/`, `Classes/Updates/`
- `Configuration/` `Services.yaml`, `Services.php`, `Icons.php`, `ExpressionLanguage.php`, `TCA/`, `Sets/`, `Yaml/` (form setup)
- `Resources/` templates, language files, public assets
- `Tests/Unit/`, `Tests/Functional/`, `Tests/Acceptance/` (Codeception, `codeception.yml`)
- `Build/checks/` configs for PHP CS Fixer, PHPStan, Rector, dependency analyser, TypoScript lint, editorconfig lint
- `Build/tests/` PHPUnit configs, `Build/environment/` test instance config, `Build/docs/` docs rendering, `Build/release/` release tooling
- `Documentation/` reStructuredText docs, including the contribution guide (`Documentation/Contributing/Index.rst`)
- `.ddev/` DDEV setup with Selenium Chrome and `init-typo3` for acceptance tests

## Development commands

Run everything in DDEV: `ddev composer install`, then `ddev composer <script>`.

```bash
composer fix          # composer normalize, editorconfig, php-cs-fixer
composer check        # deps, refactor (dry-run), static, style
composer check:deps
composer check:static # PHPStan
composer check:refactor
composer check:style  # composer, editorconfig, php, typoscript
composer refactor     # Rector, changes files
composer docs         # build and open the documentation
composer ci           # validate, audit and all checks, as run in CI
```

Single fixers: `fix:composer`, `fix:editorconfig`, `fix:php`. Single style checks: `check:style:composer`, `check:style:editorconfig`, `check:style:php`, `check:style:typoscript`.

## Testing

```bash
ddev composer test                  # functional and unit, no coverage
ddev composer test:unit
ddev composer test:functional
ddev composer test:acceptance       # runs .ddev/commands/web/init-typo3, then Codeception
ddev composer test:coverage         # acceptance, functional and unit with XDEBUG_MODE=coverage, then merged
ddev composer test:merge-coverage   # reports in Build/tests/coverage/
```

CI runs unit and functional tests on PHP 8.2 to 8.5 and TYPO3 13.4 and 14.3 with the lowest and highest dependencies. Code coverage runs in a separate DDEV-based CI job. Checks run on PHP 8.2.

## Code style and static analysis

- Follows the TYPO3 coding standards (`typo3/coding-standards`) through `eliashaeussler/php-cs-fixer-config`, config in `Build/checks/.php-cs-fixer.php`
- PHPStan level 8 (`eliashaeussler/phpstan-config`, baseline in `Build/checks/phpstan-baseline.neon`)
- Rector (`eliashaeussler/rector-config`, `ssch/typo3-rector`), config in `Build/checks/rector.php`, kept out of `check` except as dry-run
- `composer-dependency-analyser`, `helmich/typo3-typoscript-lint` (warnings fail) and `editorconfig-cli` are part of `check`
- Use `declare(strict_types=1);` in PHP files

## Git workflow

- Commit format: `<type>: <description>` with type one of `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`
- Single-line messages, no co-author trailers
- One commit per logical change, open a pull request against `main`
