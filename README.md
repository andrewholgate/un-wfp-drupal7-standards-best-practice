# UN World Food Programme: Drupal 7 Standards & Best Practice

## The United Nations World Food Programme guide to developing Drupal 7 projects.

Author : Andrew Holgate | Date : December 10, 2014 | Version : v0.9.0

# About this Document

The purpose of this guideline is to ensure that all Drupal 7 projects follow the same best practices.

This document contains a list of modern, sensible Drupal and development practises that must be followed in order to be UN WFP compliant. Each topic is covered briefly with relevant resource links for additional clarification and examples.

# Project Expectations

## Definition of "Done"

A ticket or task is only considered "done" when the changes

1. Adhere to this guidelines.
1. Successfully pass all tests
1. Deployable through an automated process into a production-identical environment.

## Realistic Task Completion Estimates

All timings for task completion must include all of the guidelines outlined in this document. Non-compliant code can only be committed to the repository when it is specifically requested before the task begins, it is clearly defined what part of the guideline will not be followed and a compliance completion roadmap is agreed upon.

It is the responsibility of each developer in the project team to understand and be able to implement the best practices outlined in this document.

The project must function correctly under all of the following circumstances:

* Using proxy forwarding, therefore HTTP calls must use [drupal_http_request()](https://api.drupal.org/api/drupal/includes%21common.inc/function/drupal_http_request)
* Using PHP accelerators, such as [APC](http://pecl.php.net/package/APC) and [Zend OPcache](http://pecl.php.net/package/ZendOpcache)
* Using in-memory caching, such as [MemCache](http://memcached.org/) and [Redis](http://redis.io/).
* Using an SMTP mail server.
* Using both http and https protocols, therefore no-hard coding of internal paths.
* Multilingual project, therefore [t()](https://api.drupal.org/api/drupal/includes%21bootstrap.inc/function/t) must be used for all user visible text, passing dynamic variables through as a parameter.
* Desktop, tablet and mobile devices, therefore a mobile or responsive theme is required.
* WFP standard browsers: Internet Explorer 8, Firefox and all modern browsers.

# Development Workflow

## Quality Assurance

All code pushed into any branch of a repository must strictly follow the best practice outlined in this document. Each commit will be carefully reviewed and evaluated by internal QA and any code found to be non-compliant will be rejected.

## Acceptance Testing

Initial Acceptance Testing is performed on successfully deployed changes on the staging server, using a current clone of the production system (database, codebase, files directory, etc).

## Automated Deployment

We take an "Automate everything approach": All deployments of project changes must be automated and function without any manual intervention, such as changes via the admin UI.

The deployment script is:

```
# Pull all code changes from the repo and update dependencies.
git pull origin master && composer update --no-dev
# Run all database updates and revert all project Features.
drush updatedb -y && drush features-revert-all -y
# Clear all Drupal caches.
drush cache-clear all
```

To achieve automated deployment, the [hook_update_N()](https://api.drupal.org/api/drupal/modules%21system%21system.api.php/function/hook_update_N) hook should be used extensively, including the installation and deinstallation of modules as well as removal of fields and variables, etc.

Once successfully deployed, all project Features should be in the default state.

## Code Review and Testing

The following tests must pass before being committed to the repository.

* [PHP_CodeSniffer](http://pear.php.net/PHP_CodeSniffer) to ensure the PHP, JS and CSS code follows standards.
 * [Coder Sniffer](https://www.drupal.org/project/coder) to test for Drupal standards.
 * [Drupal 7 Security Audit](https://github.com/Pheromone/phpcs-security-audit).
 * [DrupalPractice sniffer](https://www.drupal.org/project/drupalpractice).
 * [DrupalStrict](https://github.com/andrewholgate/drupalstrict), including cyclomatic complexity and nesting maximums.
* [PHP Mess Detector](http://phpmd.org/) to identify potential PHP problems.
* [PHP_Depend](http://pdepend.org/) to identify parts of the project where a code refactoring should be applied.
* [PHP Copy/Paste Detector](http://github.com/sebastianbergmann/phpcpd) to identify duplicated code.

All code review tests can be run from the project root by executing:

```
./scripts/review.sh
```

We use [PHPUnit](https://phpunit.de/) for unit testing. Important custom functionality of projects must always include Unit Tests.

## Pre-commit Checklist

1. Have you exported all database changes to the relevant Features, including all variables, configurations, permissions, cache settings, tags and descriptions?
1. Are the versions and dependencies correct for the Features?
1. Does the code and architecture adhere to the guidelines?
1. Is all user-visible text being translated using the t() function?
1. Are there enough code comments and are they sufficiently clear and verbose?
1. Have you run Coder and other PHP tools for testing to check against the Drupal Coding Standards for spacing, basic security, etc?
1. Has it been done in such as manner as to allow for automatic deployment?
1. Have you manually reviewed the code diff to ensure that nothing missing or extra being committed?
1. Has it been peer reviewed?

If all developers follow the best practice, the entire project code base will look as if it's been written by one developer, such as seen in [Drupal core](http://cgit.drupalcode.org/drupal/tree/modules/node/node.module?h=7.x#n681).

# Environments

## Minimum Required Setup

* A development server for current development.
 * All [error reporting must be shown](https://www.drupal.org/node/1056468) and logged.
 * Logging of all [MySQL slow queries](http://dev.mysql.com/doc/refman/5.5/en/slow-query-log.html) over 1 second.
* A staging server for performing deployments.
 * All error reporting, caching and aggregation must be turned on.
 * Logging of all MySQL slow queries over 1 second.

Both servers must be accessible via an [htdigest login and password](http://httpd.apache.org/docs/2.4/programs/htdigest.html) and use the [Environment Indicator](https://www.drupal.org/project/environment_indicator) module for environment and commit identification. The HTTP server and MySQL slow query log files should also be accessible via a URL.

All environment specific configurations must be stored in the non-revisioned [settings.local.php](https://api.drupal.org/api/drupal/sites!example.settings.local.php/8) file. Configurations such as database credentials, caching settings, site_mail variable, smtp server variables, etc. Sensitive data must never be stored inside of the code repository.

During development, content must be included in order to fully evaluated the process. In the case of real data being unavailable, test data should be generated using the [Devel](https://www.drupal.org/project/devel) module.

Unless otherwise specified, current recommended, stable versions must be used, such as for contributed modules, [PHP](https://www.drupal.org/requirements), [drush](https://github.com/drush-ops/drush#drush-versions), etc.

# Drupal Project Structure

The WFP Drupal project template must be used for all Drupal projects. The project template is based on the [Lullabot Drupal Boilerplate](https://github.com/Lullabot/drupal-boilerplate). WFP projects use Composer for project dependency management, including contributed module and themes as well as 3rd party library dependencies.

## Project Directory Structure

```
project_name/
release/
 v0.9.5/
 ..
 v1.0.0/
 .git/
 build/
 code/
 composer.json
 error.log -> project_name/logs/error.log
 slow_queries.log -> project_name/logs/slow_queries.log
 ..
current -> release/v1.0.0/build
logs/
 error.log -> /var/logs/apache/project_name.errors.log
 slow_queries.log -> /etc/mysql/slow_queries.log
settings.local.php
```

# Drupal Project Architecture

## Required Drupal Contributed Modules

* Features
* Strongarm
* Universally Unique ID
* UUID Features
* Entity API
* Diff
* Chaos Tools
* Google Analytics
* Advanced CSS/JS Aggregation

## Use Declarative Architectural Design

Before development begins, the architecture must be declared and agreed upon using the [Drupal Configuration Spreadsheet Standard](https://github.com/sreynen/Drupal-Configuration-Spreadsheet-Standard).

## Contributed Module Selection

Only install the essential contributed modules to achieve the functionality required. Modules which are not used on the project must be uninstalled immediately.

For Drupal 7 [Future-proof your Drupal 7 site](https://www.drupal.org/node/2287495) contains a list of recommended modules which should be selected for future maintenance and compatibility.

For example:

* The [Entity API](https://www.drupal.org/project/entity) and modules must be used along with the [Entity metadata wrappers](https://www.drupal.org/documentation/entity-metadata-wrappers) for accessing and manipulating entities rather than the standard Drupal approach.
* When using Date module, the Date (ISO format) must always be used and never Date or Date (Unix timestamp).

## Multilingual Projects

Projects requiring more than one language must install the base language the base language to be used as generic placeholders (eg. `form:username`), on top of which other languages are added, such as English, French, etc.

# Drupal Features Development

WFP projects make heavy use of [Features](https://www.drupal.org/project/features) to create self-contained collections of Drupal entities which satisfy a certain use-case.

All Features must follow the [Kit Specification](https://www.drupal.org/project/kit) and, when achieved, be declared [Kit Compliant](http://cgit.drupalcode.org/kit/tree/kitf.txt?id=c3717385435a4e9c59cb1ffa708a62628b0d6530#n54) by adding the following in the `.info` file

```
 kitf = "1.0-draft"
```

Each Feature must contain all relevant content types, base fields, instance fields, views, blocks, contexts, permissions, variables, image presets, taxonomies, menus, breadcrumbs, path configurations, descriptions, tags etc.

All project specific code being put inside of the `.module` file of Features.

Correct module dependencies must be assigned to each Features.

All variables and structures created for a Feature must have a corresponding deinstallation created in the Features [`.install`](https://www.drupal.org/node/876250) file.

## Development of Custom Modules

Custom module development is only required for functionality to be reusable across multiple projects. All modules must include a [composer.json](https://getcomposer.org/doc/04-schema.md#the-composer-json-schema) file.

# Drupal Development Standards and Best Practice

All developers must strictly follow the Drupal Best Practices, including:

* [Drupal Coding Standards](https://www.drupal.org/coding-standards)
* [Programming best practices](https://www.drupal.org/node/287350)
* Functions must be kept [short and easily readable](https://www.drupal.org/node/299074).
* [Write secure code](https://www.drupal.org/writing-secure-code).
* Must use appropriate [Drupal core hooks](https://api.drupal.org/api/drupal/includes!module.inc/group/hooks/7) when dealing with core.
* [CSS standards](https://www.drupal.org/node/1886770).
* [Web standards and accessibility](https://www.drupal.org/node/769692).
* [Module documentation](https://www.drupal.org/node/161085).

All standards must be implemented before code changes are committed to the repository.

## Project Security and Audits

Security upgrades for core and contributes modules much be applied within 6 hours of being announced.

In each case a security vulnerability is discovered, it must be evaluated using the [Your Drupal site got hacked. Now what?](https://www.drupal.org/node/2365547) guide. The preferred method to deal with serious threats is the rollback approach.

Project audits must be run on a regular basis on a clone of the production database and file system, using [Site Audit](https://www.drupal.org/project/site_audit), [Security Review](https://www.drupal.org/project/security_review), [Hacked!](https://www.drupal.org/project/hacked) and Diff modules via the following drush command from the root of the project:

```
./scripts/audit.sh
```

## Consistent Layout Approach

A consistent and clear approach must be taken to [managing page and content layouts](http://www.slideshare.net/mobile/davexoxide/drupal-blocks-vs-context-vs-panels) which is scalable for future project development such:

* Page Layouts: [Context](https://www.drupal.org/project/context), [Panels](https://www.drupal.org/project/panels), [Panels Everywhere](https://www.drupal.org/project/panels_everywhere).
* Content Layouts: Panels , [Panelizer](https://www.drupal.org/project/panelizer), [Display Suite](https://www.drupal.org/project/ds) , [Entity view modes](https://www.drupal.org/project/entity_view_mode)

Using the standard Drupal theme regions and Blocks is discouraged.

## No Code or Functionality Redundancy

The [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) approach must be adhered to in all aspects of the project, including custom code and module selection.

## No Hard-Coded Variable Values

No hard-coding of potentially variable values is permitted, such as:

* The domain of internal links, eg. use `/content` rather than `http://www.wfp.org/content`
* URL Paths, eg. `/homepage` might change but `/node/2` is a unique identifier.

All custom modules, Features and theme must follow the [Drupal release naming conventions](https://www.drupal.org/node/1015226).

## Database Queries

When appropriate, database queries must use the [EntityFieldQuery](https://api.drupal.org/api/drupal/includes!entity.inc/class/EntityFieldQuery) class and never make calls using non-Drupal recommended approaches.

## Patches

All project patches be stored in the `patches/` directory of the project codebase with their necessary information such as links to existing drupal.org discussion threads.

Developers must [submit patches for Drupal core or contributed modules](https://www.drupal.org/patch/submit) to drupal.org with the expectation that the will be to the relevant issue queue with the idea that the functionality will be included in a future release.

Patches must be applied using the [Composer Patches Plugin](https://github.com/netresearch/composer-patches-plugin).

Workarounds to bug or issues must be added to the code comments prefixed with a @todo and include a description and a link to the official issue report.

# Drupal Theme Development

The role of the theme is purely visual, it is for adding extra styling and markup to the output.

> "Since a module is most intimately familiar with its own data and functionality, it's the module's responsibility to provide the default theme implementation. As long as the module uses the theme system properly, a theme will be able to override any HTML and CSS by hot-swapping its own implementation for the module's implementation."
-- [Drupal 7 Module Development Theme Development](https://www.packtpub.com/books/content/drupal-7-module-development-drupals-theme-layer)

Business logic and function calls are never allowed in `template.php` or `.tpl.php` template files, whether in the theme or in custom modules.

All Themes must follow the [Kit Specification](https://www.drupal.org/project/kit) and be Kit Compliant.

## WFP Base Theme

WFP branded projects must start with the WFP Base theme with project specific customisations being added to a new project [sub-theme](https://www.drupal.org/node/225125).

# Caching & Performance

All code and architectures must implement an appropriate cache state and function correctly with Page and Basic caching is enabled, including for:

* Drupal core caching: Page, [Block](https://api.drupal.org/api/drupal/includes!common.inc/group/block_caching), Form, Menu, Path, Field, etc.
* Contributed module caches such as Views, Panels, Entity API, etc.
* Caching systems such as Boost, AuthCache, Page Manager, MemCache, Redis, Varnish, etc.

Caching expiration must be set for all Blocks, Views displays, etc.

When appropriate, [Render cache](https://www.drupal.org/project/render_cache) and/or [Entity Cache](https://www.drupal.org/project/entitycache) must be used.

> "Never do something time consuming twice if you can hold onto the results and re-use them."
-- [A Beginner's Guide to Caching Data in Drupal 7](https://www.lullabot.com/blog/article/beginners-guide-caching-data-drupal-7).

Must make effective use of [drupal_static()](https://api.drupal.org/api/drupal/includes%21bootstrap.inc/function/drupal_static/7) for persistent caching within a single page request and [cache_get](https://api.drupal.org/api/drupal/includes%21cache.inc/function/cache_get/7)() and [cache_set()](https://api.drupal.org/api/drupal/includes%21cache.inc/function/cache_set/7) for persistent caching across multiple pages.

All code must work with Drupal core JS and CSS optimisation activated as well as for the module [Advanced CSS/JS Aggregation](https://www.drupal.org/project/advagg).

# Revision Control

All [git tags](http://git-scm.com/book/en/v2/Git-Basics-Tagging) should follow the development roadmap as closely as possible, using the [Semantic Versioning](http://semver.org/) standards for tag naming.

The revision control history is as important as the code itself, therefore commit messages must be verbose and clearly describe the changes as well as being association with a relevant ticket.

View the page at: http://andrewholgate.github.io/un-wfp-drupal7-standards-best-practice/
