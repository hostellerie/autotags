# Autotags 1.2.0 — Modernization roadmap

Status: **planned** · Development branch: `develop_1.2.0` · Baseline: official Autotags 1.1.4.

## Objectives and scope

Modernize the Autotags plugin for Geeklog 2.2.2 while preserving existing autotags, their replacement content and access permissions. Fix defects in the plugin at their source: **do not add Eclipse-specific workarounds**. Follow the project's no-patchwork principle: prefer small, maintainable, testable root-cause fixes.

This is a roadmap, not a claim that the tasks below have been implemented or validated. Keep version and release notes aligned with changes actually delivered.

## P0 — Restore reliable editing and correct form semantics

- [ ] Reproduce and diagnose the failure to open/edit existing autotags when upgrading an installation from 1.1.0 to 1.1.4/1.2.0 on Geeklog 2.2.2. Record exact URL, response, PHP error log and theme independence before changing code.
- [ ] Fix form control identifiers **in Autotags** (`templates/default/admin/autotags.thtml` and any other delivered theme templates). Current default template contains `name="tag"` and `name="description"` but **no `id` attributes** on these inputs. Verify whether duplicate IDs are generated elsewhere in the rendered page. Introduce unique IDs (e.g. `autotags-tag` and `autotags-description`) and matching `<label for="...">` associations; check all other form controls for duplicate/missing IDs. Do not infer duplicate IDs in this source template where none exist.
- [ ] Verify proper HTML escaping for user-editable tag, description, and replacement fields (including quotation marks, angle brackets, UTF-8 and PHP snippets), retaining exact saved replacement semantics.
- [ ] Verify add/edit/save/cancel/delete, enable/disable, closed tags, HTML replacements and PHP replacements, including error and success messages.
- [ ] Confirm token/CSRF validation, authorization on **each** mutating action, and permissions for ordinary users versus administrators.
- [ ] Test the form with the default theme and Eclipse, on desktop and mobile. Eclipse must not need plugin-specific CSS/JS or field-ID patches.

**Acceptance:** each label targets exactly one control; no duplicate IDs in rendered markup; keyboard/assistive-technology interaction is correct; edit/save cycles preserve autotag content and permissions; no new PHP warnings or SQL errors.

## P1 — Modernize administration UI

- [ ] Replace presentation tables and inline styling in admin templates with semantic, responsive markup and clear sections for tag definition, replacement, PHP settings and permissions.
- [ ] Keep markup and CSS owned by the plugin, using Geeklog conventions; maintain compatibility with both default and Eclipse themes without theme-specific hacks.
- [ ] Improve the listing (readable status, actions, empty states and accessible edit links); use explicit, stable action values rather than comparing localized submit-button text where practical.
- [ ] Provide meaningful labels, hints, focus states and confirmation for destructive operations.
- [ ] Verify template lookup through `CTL_plugin_templatePath('autotags', 'admin')` for available theme variants.

**Acceptance:** responsive and keyboard-accessible admin interface; no regression in permissions or CRUD behavior.

## P1 — Compatibility, upgrades and security

- [ ] Audit PHP 8.1 and PHP 8.3 warnings/errors, deprecated or undefined variables, SQL escaping/parameter handling and output encoding; document any PHP 5.6 constraints supported by Geeklog 2.2.2 before making compatibility claims.
- [ ] Audit PHP-based autotag execution and validation: permissions, syntax errors, error handling, trust boundaries and stored content. Avoid altering intended behavior without a migration and explicit tests.
- [ ] Review `plugin_upgrade_autotags()`, `install_updates.php` and SQL updates for upgrades from **1.1.0 and 1.1.4**, preserving data and permissions without uninstall/reinstall.
- [ ] Validate the actual minimum Geeklog version and update plugin metadata only after tests.

**Acceptance:** backed-up legacy installation upgrades successfully; all autotags remain usable; plugin metadata matches the running code; no critical security regressions.

## P2 — Internationalization, documentation and release

- [ ] Audit language files for missing/unused keys and placeholder parity; keep UTF-8 intact. Prioritize English and French and document other shipped languages.
- [ ] Document installation, upgrading, backup/restore, PHP-based autotag security and troubleshooting for failed edits.
- [ ] Add automated static checks where practical (PHP syntax, template IDs/label associations, language placeholders) plus manual Geeklog 2.2.2 smoke-test checklist.
- [ ] Update `README.md`, `Changelog`, `INSTALL` and version metadata consistently.
- [ ] Prepare a focused PR to upstream `Geeklog-Plugins/autotags` after acceptance criteria and upgrade tests pass. Never present a branch as PR-ready or release-ready before verification.

## Suggested delivery order

1. Capture reproducible failures and fix form IDs/labels at source.
2. Validate legacy upgrade and editor CRUD/security.
3. Modernize admin UI and re-test with default + Eclipse.
4. Complete PHP compatibility, translations, documentation, automated checks.
5. Review diff, verify upgrades, then prepare PR and release notes.

## Explicit non-goals

- No changes to the Eclipse theme to compensate for plugin markup defects.
- No uninstall/reinstall strategy that discards existing autotags.
- No unrelated Geeklog core changes without a demonstrated core-level defect.
