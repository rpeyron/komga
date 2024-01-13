This is a fork of [gotson/komga](https://github.com/gotson/komga) with the following features:
- Add sort option using file date, and default sort option on libraries ([closed PR](https://github.com/gotson/komga/pull/1383))
- Add basic pan & zoom feature in the web pdf viewer

Additional instructions to build & develop locally below

### Build locally

#### Common steps

- create ~/.jreleaser/config.properties
```
JRELEASER_GITHUB_TOKEN=no
```

- build at least once komga: `./gradlew komga:build` 
- build & copy UI: `./gradlew prepareThymeLeaf` 
- build package: `./gradlew jreleaserPackage` 

Build docker image with one of the methods below and redeploy docker with this image


#### Build docker without buildx

- find how to pass TARGETARCH, or replace it by amd64 in komga/docker/Dockerfile.tpl

- build docker image: `docker build build/jreleaser/package/komga/docker/  -t komga:custom `  

One-liner:
```sh
./gradlew komga:build && ./gradlew prepareThymeLeaf && ./gradlew jreleaserPackage && docker build -t komga:rp-test build/jreleaser/package/komga/docker/
```

#### Build docker with build

- Install buildx, see official documentation: https://docs.docker.com/engine/install/ , or in debian bookworm, you can declare the debian repository with the script https://docs.docker.com/engine/install/debian/ and only install the buildx plugin with `apt install docker-buildx-plugin` (or install it manually)

- build with buildx `docker buildx build --platform linux/amd64 -t komga:custom build/jreleaser/package/komga/docker/`

One-liner:
```sh
./gradlew komga:build && ./gradlew prepareThymeLeaf && ./gradlew jreleaserPackage && docker buildx build --platform linux/amd64 -t komga:custom build/jreleaser/package/komga/docker/
```





----


[![Open Collective backers and sponsors](https://img.shields.io/opencollective/all/komga?label=OpenCollective%20Sponsors&color=success)](https://opencollective.com/komga) [![GitHub Sponsors](https://img.shields.io/github/sponsors/gotson?label=Github%20Sponsors&color=success)](https://github.com/sponsors/gotson)
[![Discord](https://img.shields.io/discord/678794935368941569?label=Discord&color=blue)](https://discord.gg/TdRpkDu)

[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/gotson/komga/tests.yml?branch=master)](https://github.com/gotson/komga/actions?query=workflow%3ATests+branch%3Amaster)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/gotson/komga?color=blue&label=download&sort=semver)](https://github.com/gotson/komga/releases) [![GitHub all releases](https://img.shields.io/github/downloads/gotson/komga/total?color=blue&label=github%20downloads)](https://github.com/gotson/komga/releases)
[![Docker Pulls](https://img.shields.io/docker/pulls/gotson/komga)](https://hub.docker.com/r/gotson/komga)

[![Translation status](https://hosted.weblate.org/widgets/komga/-/webui/svg-badge.svg)](https://hosted.weblate.org/engage/komga/)

# ![app icon](./.github/readme-images/app-icon.png) Komga

Komga is a media server for your comics, mangas, BDs, magazines and eBooks.

#### Chat on [Discord](https://discord.gg/TdRpkDu)

## Features

- Browse libraries, series and books via a responsive web UI that works on desktop, tablets and phones
- Organize your library with collections and read lists
- Edit metadata for your series and books
- Import embedded metadata automatically
- Webreader with multiple reading modes
- Manage multiple users, with per-library access control, age restrictions, and labels restrictions
- Offers a REST API, many community tools and scripts can interact with Komga
- Download book files, whole series, or read lists
- Duplicate files detection
- Duplicate pages detection and removal
- Import books from outside your libraries directly into your series folder
- Import ComicRack `cbl` read lists

## Installation

Refer to the [website](https://komga.org/docs/category/installation) for instructions.

## Documentation

Head over to our [website](https://komga.org) for more information.

## Develop in Komga

Check the [development guidelines](./DEVELOPING.md).

## Translation

[![Translation status](https://hosted.weblate.org/widgets/komga/-/webui/horizontal-auto.svg)](https://hosted.weblate.org/engage/komga/)

## Sponsors

[![Jetbrains_logo](./.github/readme-images/sponsors-jetbrains.png)](https://www.jetbrains.com/?from=Komga)

## Credits

The Komga icon is based on an icon made by [Freepik](https://www.freepik.com/home) from www.flaticon.com
