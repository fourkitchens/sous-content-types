# Sous Content Types Drupal Recipe

Provides a Page content type with a rich-text Body, SEO metadata, teaser media
and text, tags, and publication scheduling.

## Supported baselines and dependencies

Use the supported Minimal baseline in the [platform guide](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/PLATFORM.md),
with this recipe's Composer dependencies installed. Sous Admin is optional and
can be applied before or after Sous Content Types. This recipe explicitly installs the modules used by
the Page form, including Publication Date, Field Group, Scheduler, Tagify, Node,
Text, and Media Library.

Sous Formats and Sous Media are required recipes and are applied automatically.
Sous Media supplies the image media type and taxonomy/media configuration; it
also installs Media Library, Taxonomy, and Field Group. This recipe declares
Field Group's Composer dependency because the Page form uses it directly.
The dependency recipes are not replacements for Sous Admin, and Sous Admin is
not required to make a Page editable or renderable.

Existing configuration with the same values is accepted even when its generated
UUID or default configuration hash differs. Other conflicting configuration,
including a differently configured existing Page type, is rejected. Applying
this recipe is not a migration for a Standard profile or an existing site's
content model.

## Install and apply

These examples select the current unreleased implementation on `dev-main`.
Run them in a configured consuming site, not in this recipe source checkout.
[Setup and recipe prerequisites](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/SETUP.md) define the required
repositories, installer paths, core unpacking plugin, and explicit root stability
exceptions. Use reviewed stable tags once the coordinated releases exist.

In a Sous project:

```bash
ddev composer require fourkitchens/sous-content-types:dev-main
ddev install-recipe fourkitchens/sous-content-types
```

Without the Sous DDEV command, use a consuming project configured as described
in [setup](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/SETUP.md) and run from its root:

```bash
ddev drush recipe /var/www/html/web/recipes/sous-content-types
ddev drush cache:rebuild
```

Use the [current native unpacking workflow](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/SETUP.md) before application.

## Authoring and composition

Create a Page at `/node/add/page`, enter its title and required SEO Title, and
write rich text in the Content tab's Body field. Saving a published Page renders
that Body with Drupal's text formatter. The teaser display uses the dedicated
teaser media and text fields.

Sous Paragraphs and the starter's bundled native Layout Builder option are
alternative compositions built on this recipe. Apply either to the minimal
baseline or after this recipe; Sous Content Types is applied first. The core
Body field remains available alongside Paragraphs content and as a field block
in Layout Builder. The bundled core recipe retains the visible Body formatter
as a field block and enables per-Page overrides and native Basic inline blocks.
Do not reapply the base content recipe after composition has customized its
displays: strict configuration matching intentionally protects those changes.

The similarly named public `fourkitchens/sous-layout-builder` package is not
the starter's bundled option and is outside the supported combination. The
owned recipe is `web/recipes/sous-layout-builder` in the generated starter; it
uses core Layout Builder without Gin Layout Builder.

## Page regression check

In a disposable installation with this recipe applied, before adding a
composition recipe, run from the project root:

```bash
ddev drush php:script web/recipes/sous-content-types/tests/verify-page.php
```

This builds the actual Page add/edit forms, validates and saves a temporary
published Page, reloads and edits it, and verifies rendered Body content. It
deletes the test Page afterward.

To verify real HTTP form submission as well, run from the disposable project's
root, replacing `BASE_URL` with that site's full URL:

```bash
SOUS_DISPOSABLE_TEST=1 python3 web/recipes/sous-content-types/tests/verify-authoring.py BASE_URL --uid=2
```

Use `--uid=2` for a fresh Sous installation's active Chef account; bootstrap
deliberately blocks user 1. On an independent Minimal installation with an active
administrator, select that account's UID instead.

This uses Python 3 and DDEV to authenticate, submit the actual Page add/edit
forms with cookies and CSRF tokens, and check saved and rendered content. It
also deletes its temporary Page. Composition combinations are separate
integration checks.

HTTPS certificate verification is enabled. If Python does not trust the local
DDEV certificate authority, pass `--ca-file "$(mkcert -CAROOT)/rootCA.pem"`;
the shared integration runner selects that CA automatically when available.

After applying Sous Paragraphs, or enabling the core `layout_builder` module on
the default content model, run this check in the disposable site:

```bash
ddev drush php:script web/recipes/sous-content-types/tests/verify-composition.php
```

It checks that Body remains available, saves and reloads composed content, and
verifies both Body and Paragraphs output or a core Body field block. The Layout
Builder check temporarily changes the Page display and restores it afterward;
both modes delete their test content. This checks core Layout Builder, not the
currently incompatible Gin Layout Builder package described above.

## Development, integration, and releases

This repository is reusable recipe source, not a generated Drupal application.
Its Composer requirements and `recipe.yml` describe installation defaults; the
consuming project owns its runtime, complete configuration export, custom
theme, and application deployment. Requiring `dev-main` downloads remote source;
it does not test uncommitted sibling checkout changes.

Use the shared documentation instead of maintaining another setup procedure:

- [Supported runtime and package relationships](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/PLATFORM.md)
- [Fresh installation, contributor clones, restoration, and recovery](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/SETUP.md)
- [Sibling-source development and integration checks](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/INTEGRATION.md)
- [Configuration ownership and existing-site upgrades](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/MAINTENANCE.md)
- [Frontend development and production assets](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/BUILD_TOOLS.md)
- [Production deployment](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/DEPLOYMENT.md)
- [Coordinated releases and Conventional Commits](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/RELEASES.md)
- [Maintenance ownership and review cadence](https://github.com/fourkitchens/sous-drupal-project/blob/main/docs/OPERATIONS.md)

Recipe release tooling runs with the Node/npm versions pinned in `.nvmrc` and
`package.json`; Node is not required to execute a recipe in Drupal. The private
npm package publishes no browser assets or npm release. The thin workflow calls
the shared integration gate before the guarded GitHub publisher. Its central
workflow must be rolled out first; source configuration is not proof of a live
GitHub run or release. See the release guide for actual tag state, stable-tag
adoption, and explicit development exceptions.

Package updates do not automatically change configuration already applied to a
site. Preserve its complete export and adopt changes through reviewed native
configuration migrations and project-owned overrides. Do not routinely replay
the entire recipe on a customized site.

## License

Sous recipe source is licensed under [GPL-2.0-or-later](LICENSE). Dependencies
retain their own license terms; see the shared license policy in the platform
guide.
