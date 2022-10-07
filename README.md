# Overview

**TL;DR**: Learn about layer caching in docker: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache

The purpose of this repo is to demonstrate the time difference between building a docker image that does not take advantage of layer caching vs. the time it takes to build a docker image that does take advantage of layer caching. Although these demos will be using Node.js and Yarn, the same optimization could be done for projects in other languages.

Note that this caching does require that the builds be performed on the same host machine. Using these caching techniques locally is pretty straight-forward, but may take more configuration depending on how your CI/CD pipeline is setup.

# Requirements

- Docker installed and accessible in the command line

# Running the demo

## Node API (simple project)

Since this app is small and only has a few dependencies the difference is not that large.

Change to the node-api project directory
`cd packages/node-api`

### First Build

Each of the two following builds should take roughly the same amount of time. Since this is your first time building each of the images, none of the layers should be cached (with the exception of `node:16-alpine` if you have used that before).

The fast build may be slightly faster since `node:16-alpine` gets cached when building the slow build. You can tell that a layer is cached because it will say something like `=> CACHED [1/3] ...`

1. Build the non-optimized dockerfile:
`docker build . -f Dockerfile.slow -t node-api-cache-layers-slow`

2. Build the optimized dockerfile:
`docker build . -f Dockerfile.fast -t node-api-cache-layers-fast`

### Second Build

Each of these builds should again take roughly the same amount of time. They should be almost instant since all layers are cached due to the previous build.

Build each of the dockerfiles again

1. `docker build . -f Dockerfile.slow -t node-api-cache-layers-slow`
2. `docker build . -f Dockerfile.fast -t node-api-cache-layers-fast`

### Build After A Change

This is where the caching optimization takes effect. Before building again we will first make a change to one of the src files.

The Dockerfile.slow file copies all source files before running yarn install. When docker copies the source files in stage 2, it will notice that some of the source files have been changed, that prevents that stage from being cached. Since the `yarn install` stage is after the copy source files stage, the yarn install stage also cannot be cached. This causes the yarn install to be run, which takes some time.

Notice that the Dockerfile.fast file copies only the package.json file before running yarn install. If there are no changes to the `package.json` file, docker can use the cached layer. This allows docker to also used the cached layer for the `yarn install` stage. Then docker copies the rest of the source files, which cannot use the cached layer since there are file changes. Notice that in this case, docker was able to use the cached `yarn install` layer which can speed up the build significantly depending on the number of dependencies.

Lets see this in action:

1. Make a change to one of the source files, for example, change the version number in ./packages/node-api/src/index.js line 8
2. `docker build . -f Dockerfile.slow -t node-api-cache-layers-slow` - notice that stage 3 (`yarn install` stage) is not cached due to the source code update
3. `docker build . -f Dockerfile.fast -t node-api-cache-layers-fast` - notice that stage 3 (`yarn install` stage) is cached, since the package.json file did not change

The Dockerfile.slow build will take some time since the `yarn install` needs to run, whereas the Dockerfile.fast build will complete almost instantly.

Starting the app (not necessary)
`docker run -d -p 3001:3000 node-api-cache-layers-{slow|fast}`

## NextJS App (bigger project)

Since this app is bigger (using Next.js, React, etc) and has more dependencies, the difference in build times will be larger.

Change to the next-app project directory
`cd packages/next-app`

### First Build

Each of the two following builds should take roughly the same amount of time. Since this is your first time building each of the images, none of the layers should be cached (with the exception of `node:16-alpine` if you have used that before).

The fast build may be slightly faster since `node:16-alpine` gets cached when building the slow build. You can tell that a layer is cached because it will say something like `=> CACHED [1/3] ...`

1. Build the non-optimized dockerfile:
`docker build . -f Dockerfile.slow -t next-app-cache-layers-slow`

2. Build the optimized dockerfile:
`docker build . -f Dockerfile.fast -t next-app-cache-layers-fast`

### Second Build

Each of these builds should again take roughly the same amount of time. They should be almost instant since all layers are cached.

Build each of the dockerfiles again

1. `docker build . -f Dockerfile.slow -t next-app-cache-layers-slow`
2. `docker build . -f Dockerfile.fast -t next-app-cache-layers-fast`

### Build After A Change

This is where the caching optimization takes effect. Before building again we will first make a change to one of the src files.

The Dockerfile.slow file copies all source files before running yarn install. When docker copies the source files in stage 2, it will notice that some of the source files have been changed, that prevents that stage from being cached. Since the `yarn install` stage is after the copy source files stage, the yarn install stage also cannot be cached. This causes the yarn install to be run, which takes some time.

Notice that the Dockerfile.fast file copies only the package.json file before running yarn install. If there are no changes to the `package.json` file, docker can use the cached layer. This allows docker to also used the cached layer for the `yarn install` stage. Then docker copies the rest of the source files, which cannot use the cached layer since there are file changes. Notice that in this case, docker was able to use the cached `yarn install` layer which can speed up the build significantly depending on the number of dependencies.

Lets see this in action:

1. Make a change to one of the source files, for example, change the "Welcom to Next.js!" heading in ./packages/next-app/pages/index.tsx line 17
2. `docker build . -f Dockerfile.slow -t next-app-cache-layers-slow` - notice that stage 3 (`yarn install` stage) is not cached due to the source code update
3. `docker build . -f Dockerfile.fast -t next-app-cache-layers-fast` - notice that stage 3 (`yarn install` stage) is cached, since the package.json file did not change

The Dockerfile.slow build will take some time since the `yarn install` needs to run, whereas the Dockerfile.fast build will complete almost instantly.

Starting the app (not necessary)
`docker run -d -p 3000:3000 next-app-cache-layers-{slow|fast}`

# Summary

As you have seen, caching layers which take time to complete can greatly reduce the amount of time that it takes to build a docker image.

You can see whether a layer was cached by looking at the layer specifier, if the layer is cached it will look something like: `=> CACHED [1/4] ...`

In this case we optimized the `yarn install` layer, but the same can be true for other languages. For example in python you can copy the `requirements.txt` file rather than all source code, then install dependencies (which will be cached as long as requirements.txt doesn't change), then copy over your source code.

The same can be done for compiled languages. However if you want to reduce image size for compiled language projects, you may prefer to use a multi-stage build: https://docs.docker.com/build/building/multi-stage/