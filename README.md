# Sources of the GLSP Website

This page hosts the sources of [eclipse.dev/glsp](https://www.eclipse.dev/glsp).
We use the [Syna](https://github.com/okkur/syna) thema for [Hugo](https://gohugo.io/)

Please check the [Syna documentation](https://about.okkur.org/syna/docs/). The Syna theme heavily works with fragments, therefore the development differs a bit from a "normal" Hugo website.

## Quick start

### Open in VS Code dev container

To avoid the need to install hugo on development machines, a VS Code dev container is provided that runs hugo in a docker container.

* Get submodules `git submodule init && git submodule update`
* Open the repository folder in VS Code
* If not installed, install Microsoft's "Remote - Containers" extension with id `ms-vscode-remote.remote-containers`.
* Open this repository in VS Code as a [Dev Container](https://code.visualstudio.com/docs/remote/containers#_quick-start-open-an-existing-folder-in-a-container):
  * Open command palette (`F1` or `Ctrl+Shift+P`)
  * Run `Remote-Containers: Reopen in Container`

### Development

* Start Hugo server via `hugo server` for development or run task `Serve Drafts`
* Build the website via `hugo` for deployment (the public folder ist deployed then) or run task `Build`

## Overview

 * `config.toml` contains the global config and menu items
 * `content/_global` contains customization for global parts of the website, for example `footer`
 * `content/_index` contains the landing page
 * `content/XYZ/*` contains each reachable page. `index.md` is necessary to declare the page exists, while `content.md` defines its contents. Additional fragments can be added / overwritten etc.
 * `static/***` contains static resources, for example images
 * `archetypes` contains templates which are used when executing `hugo new`. Not too important but easier than copy & pase.

 ## How to create a new page

  1. Either copy & paste an existing one, or execute `hugo new --kind page-bundle <NAME>`
  2. To add an entry to the menu, add a link to the page in `config.toml`

 ## Fancy elements

  Check the [Syna Fragments](https://about.okkur.org/syna/fragments/) documentation for all provided fragments and how to use them.

 ## Sorting elements

  To sort items you can use weight, a lower value typically means it is more to the top or left. However note that `weight = 0` is the same as undefined, so use at least `weight = 1`.

 ## Best practices

  Check the example site provided with Syna in `themes/syna/exampleSite`

## Deployment

Pushing to `master` builds the website and pushes the result to [`eclipse-glsp/glsp-website`](https://github.com/eclipse-glsp/glsp-website), which is served at [eclipse.dev/glsp](https://www.eclipse.dev/glsp).

Pull requests are built and deployed to the separate [`glsp-previews`](https://github.com/eclipse-glsp/glsp-previews) repository, under `glsp-website-source/pr-previews/pr-<number>/`, and are served at `https://eclipse-glsp.github.io/glsp-previews/glsp-website-source/pr-previews/pr-<number>/`. A pull request gets a preview unless it only changes `README.md`, `LICENSE`, `.vscode/` or `.devcontainer/`. A comment on the pull request tracks the deployment and links the preview next to the live website. Closing the pull request removes the preview.

The build and the deployment are split into two workflows: [`pr-preview-build.yml`](.github/workflows/pr-preview-build.yml) runs the pull request code without any secret, and [`pr-preview-deploy.yml`](.github/workflows/pr-preview-deploy.yml) only consumes the resulting artifact. The deployment uses the `GH_DEPLOY_TOKEN` secret, a token scoped to `glsp-previews` alone, so it is never reachable from code contributed in a pull request and cannot write to this repository.
