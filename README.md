# Website

We use the [Syna](https://github.com/okkur/syna) thema for [Hugo](https://gohugo.io/)

Please check the [Syna documentation](https://about.okkur.org/syna/docs/). The Syna theme heavily works with fragments, therefore the development differs a bit from a "normal" Hugo website.

## Quick start

### Open in VSCode dev container

To avoid the need to install hugo on development machines, a VSCode dev container is provided that runs hugo in a docker container.

* Get submodules `git submodule init && git submodule update`
* Open the repository folder in VSCode
* If not installed, install Microsoft's "Remote - Containers" extension with id `ms-vscode-remote.remote-containers`.
* Open this repository in VSCode as a [Dev Container](https://code.visualstudio.com/docs/remote/containers#_quick-start-open-an-existing-folder-in-a-container):
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
