# create-pageowl

Scaffold a self-hosted PageOwl project from the GitHub release matching this CLI's version. Requires Node.js 20+ and Git; the generated project uses Bun 1.4+.

Version 1.0.0 is published to the **GitHub Packages npm registry**, not npmjs.com. Publishing this small CLI to npmjs.com is planned so users can run `bunx @iamparmjeet/create-pageowl my-pageowl` without GitHub registry authentication (assuming the npm account owns the `@iamparmjeet` scope). The full PageOwl app remains private in its root `package.json` and is not an npm dependency.

## What the CLI does

`create-pageowl my-pageowl` refuses to overwrite an existing directory. It clones `https://github.com/iamparmjeet/PageOwl.git` at the tag matching its own version (for 1.0.0, `v1.0.0`), resets `d1_databases[DB].database_id` in the new `wrangler.jsonc` to an all-zero placeholder, and removes the clone's `.git` directory. It leaves the checked-in Worker, D1, and R2 names alone; provision your own resources and replace the D1 ID before deployment. It does not install dependencies, create secrets, or deploy the Worker. A missing tag or invalid release fails without leaving a prepared project. The repeatable check is `bun run scripts/create-pageowl-e2e.ts` from the repo root; its output is recorded in `docs/e2e/create-pageowl.log`.

GitHub Packages requires a GitHub personal access token (classic) with `read:packages` even for a public package. In your user-level `.npmrc`, configure the registry and token (do not commit the token):

```ini
@iamparmjeet:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_PACKAGES_TOKEN}
```

Set `GITHUB_PACKAGES_TOKEN` in your shell, then install and run with Bun:

```bash
bun add --global @iamparmjeet/create-pageowl@1.0.0
create-pageowl my-pageowl
cd my-pageowl
bun install
```

The scaffold resets the repository's D1 database ID to a placeholder, removes its Git history, and does not install dependencies or deploy anything. Read `README.md` and `docs/deploy.md` in the generated directory to provision your own D1, R2 and OAuth secrets before deploying.

Without GitHub Packages authentication, the same versioned source can be cloned directly:

```bash
git clone --branch v1.0.0 https://github.com/iamparmjeet/PageOwl.git my-pageowl
```

When cloning directly, **replace the D1 database ID** in `wrangler.jsonc` before deploying; that file in the source repository names the maintainer's existing database.

## Publishing to npmjs.com (not yet done)

1. Sign in to npmjs.com with an account that owns the `@iamparmjeet` scope, or select another available package name and update `packages/create-pageowl/package.json`. The GitHub Packages name does not reserve the npmjs.com name. Confirm the version and the matching GitHub tag both exist; never republish a changed tarball under an existing version.
2. Run `bun run scripts/create-pageowl-e2e.ts` from the repository root. From `packages/create-pageowl`, run `bun pm pack` and review the package contents: the manifest, CLI, README, and license. Keep credentials out of the package and repository.
3. Configure npmjs.com authentication **outside this repository**, for example in your user-level `.npmrc` with `//registry.npmjs.org/:_authToken=${NPM_TOKEN}` and a token supplied in the environment. If your npm config maps `@iamparmjeet` to GitHub Packages, remove or override that mapping for this publish: scoped registry settings may take precedence over `--registry`.
4. From `packages/create-pageowl`, run `bun publish --registry=https://registry.npmjs.org --access public`. Confirm the output says `Registry: https://registry.npmjs.org/` and check the package page before documenting it as available. A scoped public package needs `--access public` on first publish.

After publication, users with the matching npm scope can run `bunx @iamparmjeet/create-pageowl my-pageowl`. The shorter `bunx create-pageowl my-pageowl` requires a separate **unscoped** `create-pageowl` package on npmjs.com; it has not been published.
