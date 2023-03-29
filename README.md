# Keptn Documentation Tooling

This is our Documentation Tooling repository.

It contains:

- our [Docsy][Docsy] theme adaptions
- our Documentation Release Workflow

## Hugo Theme

[Docsy][Docsy] is a powerful Hugo theme, created and maintained by Google.

It supports multi-version documentation and it has i18n-support integrated.

Furthermore, it offers a search etc.

## Release Workflow

We are using Netlify to publish our pages.
There are 3 different types of publication:

1. pull request previews
2. development documentation aka staging (build of `main` branch)
3. official documentation aka production (build of `page` branch)

Within the navigation bar, we do have version links pointing to the different publications - if it makes sense.
For example, we are not linking from development and production to pull request previews.

### Pull request preview

- build: on each pull request with documentation changes
- build-environment: development
- config folder: [_default](./config/_default/)

The pull request preview will be generated if documentation files have been touched - this is configured in the `netlify.toml`.

This preview should help contributors to inspect their changes within our usual page release.
Furthermore, it allows reviewers to inspect the rendered documentation without building it on their own.

### Development page

- build: on each push to `main` with documentation changes
- build-environment: main
- config folder: [main](./config/staging/)

This page reflects the current development status of the documentation.
It will be built regularly and can be easily accessed.

It should allow bleeding-edge users and contributors to see the current state and help with debugging etc.

### Official documentation

- build: on each push to `page` with documentation changes
- build-environment: production
- config folder: [production](./config/production/)

This documentation set contains all released versions of KLT and is stored in an orphaned branch called `page`.

Each version has its own `docs` folder named `docs-<version>`.
Except for the latest version which will be within the `docs` folder.

Each version-specific documentation contains a `version` file containing the version string.
This is important so we do know which version it contains - specifically important for `docs` of the latest version.

`Docsy` offers a mechanism to build a version menu based on Hugo's configuration.
We extended this mechanism and enhanced it with a check for directories starting with `docs` and containing a `version` file.
For more details inspect the [layout file with adaptions](layouts/partials/navbar-version-selector.html).
This way we do not need to adapt the configuration all the time we are releasing a new version.

[Docsy]: https://www.docsy.dev/
