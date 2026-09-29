# pw-module-svg-sanitizer

Castus fork of [ryancramerdesign/FileValidatorSvgSanitizer](https://github.com/ryancramerdesign/FileValidatorSvgSanitizer). It's a ProcessWire file validator that sanitises uploaded SVG files: it strips scripts, event handlers and remote references before the file is saved.

> This README lives in `.github/` so that upstream's own [`README.md`](../README.md) stays untouched and never conflicts when we merge upstream changes. Upstream's README still describes the bundled `svgSanitize/` library; in this fork that copy has been replaced, see "Our patches".

## What this is

- **Used by:** [castusdesign/intertrain](https://github.com/castusdesign/intertrain). It runs automatically on every SVG upload to an image or file field; five image fields accept SVG. It's configured to remove remote references. **It's an XSS defence, so keep the library current.**
- **Composer package:** `castusdesign/pw-module-svg-sanitizer` (type `processwire-module`)
- **Installs to:** `public_html/site/modules/FileValidatorSvgSanitizer/` (set by `extra.installer-name`)
- **Pulls in:** [`enshrined/svg-sanitize`](https://packagist.org/packages/enshrined/svg-sanitize) `^1.0` from Packagist. Because it's a real Packagist package, Dependabot in the Intertrain repo alerts on it.

## Branches

| Branch | Contains | Rule |
|---|---|---|
| `source` | Upstream history, exactly as published | **Never commit our changes here.** It only ever fast-forwards to an upstream commit. |
| `main` (default) | `source` + `composer.json`, `.gitattributes`, this README, and our patches | Every change is its own commit, listed below |

## Current base

- **Upstream:** https://github.com/ryancramerdesign/FileValidatorSvgSanitizer
- **Version:** module version 5, commit `0db9955` ("Update to svgSanitizer 0.14.1 of svg-sanitizer library", 2021). Upstream doesn't tag releases.
- **Our release tag:** `0.0.5-patch1`. ProcessWire's integer version 5 is `0.0.5` in semver.

## Our patches

| Commit on `main` | What it changes | Why |
|---|---|---|
| "Use enshrined/svg-sanitize from Composer instead of the bundled copy" | Deletes `svgSanitize/`. In `getSvgSanitizer()`, replaces the `classLoader->addNamespace(…/svgSanitize/)` registration with an error if `enshrined\svgSanitize\Sanitizer` can't be autoloaded. Adds the `enshrined/svg-sanitize` requirement to `composer.json`. | The bundled copy is svg-sanitizer **0.14.1**, which is affected by published advisories: GHSA-fqx8-v33p-4qcc and GHSA-xrqq-wqh4-5hg2 (XSS bypass, fixed 0.15.0 and 0.16.0), GHSA-22wq-q86m-83fh (attribute sanitisation bypass, fixed 0.22.0), and stored XSS and CSS injection issues fixed in 1.0.0. The library API this module uses (`Sanitizer`, `TagInterface`, `AttributeInterface`) is unchanged from 0.14.1 to 1.0.0. |

**Can it be dropped?** Only if upstream itself switches to Composer for the library. As long as upstream bundles a copy, keep this patch, even if upstream updates the bundled version. Otherwise we lose Dependabot tracking and end up on whatever version upstream last copied in.

## How to update

### Updating the module (upstream changes)

1. **One-time setup** in your clone:

   ```sh
   git remote add upstream https://github.com/ryancramerdesign/FileValidatorSvgSanitizer.git
   ```

2. **Fetch upstream and move `source` to its new head.** Upstream has no tags, so use the commit:

   ```sh
   git fetch upstream
   git log --oneline source..upstream/master     # review what's new
   git switch source
   git merge --ff-only upstream/master
   git push origin source
   ```

3. **Merge into `main`:**

   ```sh
   git switch main
   git merge source
   ```

   - If upstream updated its bundled `svgSanitize/`, the merge brings the directory back. Delete it again with `git rm -r svgSanitize` and amend the merge.
   - If upstream changed `getSvgSanitizer()`, re-apply our loader change by hand.

4. **Tag and push.** Use the module's new integer version as semver, plus `-patchN`. For example, version 6 becomes `0.0.6-patch1`.

   ```sh
   git tag 0.0.6-patch1
   git push origin main --tags
   ```

### Updating only the svg-sanitizer library

Nothing to do here. In the Intertrain repo, run:

```sh
docker compose exec app composer update enshrined/svg-sanitize
```

If a new **major** version of the library is released, first check that `Sanitizer`, `data\TagInterface` and `data\AttributeInterface` still have the method signatures that `FileValidatorSvgSanitizer.module.php` and `FileValidatorSvgSanitizer.data.php` use. Then widen the constraint in this repo's `composer.json` and tag a new `-patchN`.

## Rolling it out in Intertrain

In the Intertrain repo:

1. Run `docker compose exec app composer update castusdesign/pw-module-svg-sanitizer`.
2. Commit the changed `composer.lock`.
3. In the admin, go to Modules > Refresh.
4. Smoke-test:
   - Upload a clean SVG to an image field that accepts SVG. It saves and displays.
   - Upload an SVG containing `<script>` or an `onload=` attribute. The saved file has it stripped.
5. Open a PR. Deploying is done by the team as usual.

## Licence

- **The module:** MIT, as upstream (see the header of `FileValidatorSvgSanitizer.module.php`).
- **The svg-sanitizer library:** GPL-2.0-or-later. It's no longer in this repo; Composer installs it.
