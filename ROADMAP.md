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

## P1 — Modernize built-in YouTube autotag

- [ ] Replace the legacy `youtube` default replacement (`<iframe width="560" height="315" ...>`) in `sql/mysql_install.php` with the following responsive, click-to-play, privacy-enhanced embed.
- [ ] Preserve the `#1` parameter substitution used in `[youtube:video_id]` and confirm it is restricted to a valid YouTube video ID before insertion into HTML/URLs.
- [ ] Make installation/upgrade behavior explicit: **new installations** receive the new default; **existing records** must not be unconditionally overwritten during upgrade because users may have customized their `youtube` autotag. Offer a documented opt-in migration or update only when the stored replacement matches the historical default exactly, with backup and tests.
- [ ] Preserve the revised explicit `rgb()` / `rgba()` colors when storing, filtering, editing and rendering the embed; check whether Geeklog's HTML sanitizer modifies color syntax.
- [ ] Test that the generated `srcdoc` survives SQL/PHP quoting, Geeklog autotag replacement, form editing and saving, and HTML rendering; validate iframe `allow` / autoplay, referrer policy, keyboard activation, privacy and fallbacks with appropriate CSP.
- [ ] Check thumbnail request behavior (`img.youtube.com` is contacted before interaction), caching/SEO and accessibility. Privacy-enhanced `youtube-nocookie.com` only applies after the user activates playback.
- [ ] Check localization of the iframe title and the play link's accessible label if a localized default is required; do not hard-code theme-specific dependencies.

Updated default replacement supplied after resolving a rendering issue (use explicit `rgb()` / `rgba()` color values in place of compact hex colors; subject to functional verification):

```html
<div style="position:relative;width:100%;aspect-ratio:16/9;overflow:hidden;background:rgb(0,0,0);">
  <iframe
    title="Lecteur vidéo YouTube"
    loading="lazy"
    referrerpolicy="strict-origin-when-cross-origin"
    allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen
    style="position:absolute;inset:0;width:100%;height:100%;border:0;"
    srcdoc="
      <style>
        *{box-sizing:border-box}
        html,body{margin:0;width:100%;height:100%;background:rgb(0,0,0)}
        a{display:block;position:relative;width:100%;height:100%;overflow:hidden}
        .thumb{width:100%;height:100%;object-fit:cover}
        .play{
          position:absolute;top:50%;left:50%;
          transform:translate(-50%,-50%);
          display:grid;place-items:center;
          width:68px;height:48px;
          border-radius:12px;
          background:rgb(204,0,0);
          color:rgb(255,255,255);
          box-shadow:0 2px 12px rgba(0,0,0,.53);
          font:32px Arial,sans-serif;
        }
        a:hover .play,a:focus-visible .play{background:rgb(238,0,0)}
        a:focus-visible{outline:3px solid white;outline-offset:-3px}
      </style>
      <a href='https://www.youtube-nocookie.com/embed/#1?autoplay=1'
         aria-label='Lire la vidéo YouTube'>
        <img class='thumb'
             src='https://img.youtube.com/vi/#1/hqdefault.jpg'
             alt=''>
        <span class='play' aria-hidden='true'>&#9654;</span>
      </a>
    "
  ></iframe>
</div>
```

**Acceptance:** responsive 16:9 thumbnail, click-to-play, unique replacement of `#1`, accessible activation, no unintended rewrites of customized records and no theme-specific workaround.

## Post-1.2.0 vision — Unified autotag discovery and insertion

Autotags should be the **single, read-only discovery catalogue for tags owned by Geeklog Core and other installed plugins**, while remaining the editor for its **own** custom tags. Do not duplicate other plugins' capabilities. MediaGallery's existing Forum media-picker flow is a useful UX reference: pick an authorized item, then insert the correct syntax automatically. Reuse or interoperate with existing picker integrations rather than writing a parallel MediaGallery browser.

### 1.3.0 — User-facing picker and guided insertion

- [ ] Provide a consistent "Insert autotag" control for supported article, static-page, Forum and comment editors and submission forms; begin with a feasibility survey of each editor type, including simple textareas and CKEditor.
- [ ] Search/filter the unified catalogue by name, description, provider, category, allowed context and permission; show the exact syntax plus a copy option where direct insertion is unavailable.
- [ ] Supply contextual forms from optional provider metadata (e.g., paste a YouTube URL; select an album via MediaGallery's picker); generate valid syntax and insert at the caret, preserving selection and drafts.
- [ ] Show only available/usable autotags to the current user; respect owner-specific selection and visibility (e.g., MediaGallery albums belonging to, or otherwise visible to, the user), and enforce permissions again server-side.
- [ ] Keep interoperability with existing Forum/MediaGallery media insertion; do not replace that navigator or introduce duplicate buttons without an integration plan.

### 1.4.0 — Administration, previews and history

- [ ] Add a unified catalogue of **Core, third-party plugin and custom Autotags** with source/provider, enabled/available status, description, syntax, examples, permissions and dependencies.
- [ ] Read-only entries for Core/third-party tags (never edit their definitions through this plugin); only Autotags-owned definitions are editable here.
- [ ] Detect **exact name collisions** before creation/save, with clear owner identification and a choice to cancel or rename; audit normalization/case-sensitivity and how Geeklog actually resolves duplicate providers. Do not claim automated semantic equivalence: optionally suggest similar purposes using descriptions and manual review.
- [ ] Add **safe preview** for tags where allowed: prefer declared static example/thumbnail or an opt-in provider preview API. Do **not** blindly execute arbitrary tag PHP, remote embeds, side effects or permission-sensitive content in the admin catalogue; sandbox or isolate render previews and offer safe fallback where unsupported.
- [ ] Add categories, duplication of custom definitions, import/export with conflict checks, and version history/rollback for locally managed autotags only.

### 1.5.0 — Provider metadata and plugin integration

- [ ] Investigate Geeklog Core tag-discovery APIs and plugin_autotags hooks, including `tagname`, `description`, `permission`, `nopermission` and `closetag`; distinguish dynamically registered providers from database-defined tags. Do not assume all plugins expose richer schemas or preview support.
- [ ] Define an **optional backwards-compatible metadata contract**, initially a local adapter, for provider name, category, help text, syntax, sample data, input schema, picker callback, preview capability and context constraints.
- [ ] Prototype opt-in adapters for MediaGallery, Downloads, Documents, Videos/Radio, FAQ and Hub; integrate with their canonical selectors and APIs, without moving their business logic into Autotags.
- [ ] Avoid arbitrary include/execute scans to discover tags; respect plugin activation, current-user visibility, provider ordering, caching and cache invalidation when plugins are enabled/disabled.
- [ ] Document ownership boundaries: Core parses and routes; providers own semantics and data; Autotags owns its custom definitions and the shared discovery/insertion UX.

### 2.0.0 — Optional advanced syntax (research only)

- [ ] Study named parameters, reusable editorial components and richer dynamic tags, conditioned on **Core parser compatibility**. Preserve existing positional `#1`, `#2`, `#3` and current syntax; avoid a conflicting second parser.

### Cross-version acceptance and test matrix

- [ ] Test discovery with no extra plugins, one third-party provider, multiple providers, deactivated providers and colliding names.
- [ ] Verify that catalogue read-only entries are never mutated and that old enabled custom tags continue rendering unchanged.
- [ ] Check admin versus contributor visibility, editor contexts, Forum/MediaGallery ownership permissions, malicious input, and previews that cannot execute privileged PHP or network actions implicitly.
- [ ] Check mobile, keyboard, screen readers, translations and compatibility with default and Eclipse themes.
- [ ] Measure performance with many providers/tags and ensure catalogue cache invalidation behaves correctly.

**Implementation boundary:** 1.2.0 remains focused on reliability, security, admin fixes and the YouTube tag. New catalogue, insertion, preview and metadata features are **planned only** for later milestones; numbering is provisional.

## P2 — Internationalization, documentation and release

- [ ] Audit language files for missing/unused keys and placeholder parity; keep UTF-8 intact. Prioritize English and French and document other shipped languages.
- [ ] Document installation, upgrading, backup/restore, PHP-based autotag security and troubleshooting for failed edits.
- [ ] Add automated static checks where practical (PHP syntax, template IDs/label associations, language placeholders) plus manual Geeklog 2.2.2 smoke-test checklist.
- [ ] Update `README.md`, `Changelog`, `INSTALL` and version metadata consistently.
- [ ] Prepare a focused PR to upstream `Geeklog-Plugins/autotags` after acceptance criteria and upgrade tests pass. Never present a branch as PR-ready or release-ready before verification.

## Suggested delivery order

1. Capture reproducible failures and fix form IDs/labels at source.
2. Validate legacy upgrade and editor CRUD/security.
3. Modernize admin UI and built-in YouTube embed; re-test with default + Eclipse.
4. Complete PHP compatibility, translations, documentation, automated checks.
5. Review diff, verify upgrades, then prepare PR and release notes.

## Explicit non-goals

- No changes to the Eclipse theme to compensate for plugin markup defects.
- No uninstall/reinstall strategy that discards existing autotags.
- No unrelated Geeklog core changes without a demonstrated core-level defect.
