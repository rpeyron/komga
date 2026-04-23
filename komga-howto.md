# Develop

Run server (remove `config-dirc/local-db*` if needed):
```
./gradlew bootRun --args='--spring.profiles.active=dev,localdb,noclaim'
```

If using `console.log` in watch mode (that MUST be removed to be able to build), in fork-ts-checker-config.js, change from 4Go to 8Go:
```
    memoryLimit: 8192,
```

Run front, in komga-webui (after `npm i`, once):
```
npm run serve
```

# Build locally

## Common steps

- create ~/.jreleaser/config.properties
```
JRELEASER_GITHUB_TOKEN=no
```

- build at least once komga: `./gradlew komga:build` 
- build & copy UI: `./gradlew prepareThymeLeaf` 
- build package: `./gradlew jreleaserPackage` 

Build docker image with one of the methods below and redeploy docker with this image


# Build docker without buildx

- find how to pass TARGETARCH, or replace it by amd64 in komga/docker/Dockerfile.tpl

- build docker image: `docker build build/jreleaser/package/komga/docker/  -t komga:custom `  

One-liner:
```sh
./gradlew komga:build && ./gradlew prepareThymeLeaf && ./gradlew jreleaserPackage && docker build -t komga:rp-test build/jreleaser/package/komga/docker/
```

And to save:
```sh
docker tag komga:rp-test komga:rp-2026-04-21
docker save -o komga-rp-2026-04-21.tar komga:rp-2026-04-21*
docker load -i komga-rp-2026-04-21.tar
```



# Build docker with build

- Install buildx, see official documentation: https://docs.docker.com/engine/install/ , or in debian bookworm, you can declare the debian repository with the script https://docs.docker.com/engine/install/debian/ and only install the buildx plugin with `apt install docker-buildx-plugin` (or install it manually)

- build with buildx `docker buildx build --platform linux/amd64 -t komga:custom build/jreleaser/package/komga/docker/`

One-liner:
```sh
./gradlew komga:build && ./gradlew prepareThymeLeaf && ./gradlew jreleaserPackage && docker buildx build --platform linux/amd64 -t komga:custom build/jreleaser/package/komga/docker/
```

Note there must be a 'origin' remote set up on your git repository (strange thing for local build, no idea why...)

