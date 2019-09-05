# Website

We use the [Syna](https://github.com/okkur/syna) thema for [Hugo](https://gohugo.io/)

Please check the [Syna documentation](https://about.okkur.org/syna/docs/). The Syna theme heavily works with fragments, therefore the development differs a bit from a "normal" Hugo website. 

## Quick start

### Prerequisites

  * Get submodules `git submodule init && git submodule update`
  * Install Hugo. You can skip step 1 & 2 when you already have brew installed
    1. `sudo apt install linux-brew-wrapper`
    2. `brew` (This will execute the first time setup)
    3. `brew install hugo`

### Development

  * Start Hugo server via `hugo server` for development
  * Build the website via `hugo build` for deployment

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


