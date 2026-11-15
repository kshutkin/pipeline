# pipeline
Generic pipeline for my repositories

Publishing uses npm trusted publishing (OIDC). No npm publishing token or
`secrets: inherit` is needed. Publishing uses the npm bundled with the current
Node.js LTS release. The build and coverage jobs do not have permission to
request OIDC tokens.

Call the workflow with these job permissions:

```yaml
jobs:
  pipeline:
    uses: kshutkin/pipeline/.github/workflows/main.yml@main
    permissions:
      contents: write
      pull-requests: write
      id-token: write
```

Keep any existing `with` inputs, such as `scope` and `has-dist`.

## Manual migration steps

1. Before a package's next release, open its npm **Settings → Trusted publishing**
   and add GitHub Actions. Set owner to `kshutkin`, repository to the calling
   repository, workflow filename to `main.yml`, and leave environment empty. Enable direct
   `npm publish`; these release scripts do not use staged publishing. Repeat for
   each package being released.
2. Commit and push the shared pipeline change to `main`, then the caller changes
   to their repositories. Coordinate these updates before running a release:
   callers use `@main`, so the shared change takes effect immediately.
3. Check GitHub **Settings → Actions → General → Allow GitHub Actions to create
   and approve pull requests** in each caller repository.
4. Run the normal Changesets release flow on `main`. An unchanged version or a
   release PR alone does not exercise publishing. Each newly added npm trust
   configuration must complete its first successful publish within two days;
   recreate it if it expires.
5. After successful OIDC releases, remove unused `NPM_TOKEN` GitHub secrets and
   revoke the corresponding npm tokens once no other workflows need them.

npm validates the calling repository and workflow, so register the package's
repository, not `pipeline`. See [npm trusted publishing documentation](https://docs.npmjs.com/trusted-publishers/)
for setup, expiry, and reusable-workflow behavior.

For unpublished packages, make the first release with interactive npm
authentication in a disposable checkout, then configure trusted publishing for
subsequent releases. npm requires a package to exist before configuring trust.
Use a disposable checkout because these packages run `pkgprn` during packing,
which rewrites package files. See [npm trust prerequisites](https://docs.npmjs.com/cli/v11/commands/npm-trust/#prerequisites).
