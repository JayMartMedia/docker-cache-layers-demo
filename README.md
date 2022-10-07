## Overview

The purpose of this repo is to show the time difference between building a docker image that does not take advantage of layer caching vs. the time it takes to build a docker image that does take advantage of layer caching.

## Requirements

- Docker installed

## Running the demo

### Node API (simple project)

Change to the node-api project directory
`cd packages/node-api`

Build the non-optimized dockerfile:
`docker build . -f Dockerfile.slow -t node-api-cache-layers-slow`

Build the optimized dockerfile
`docker build . -f Dockerfile.fast -t node-api-cache-layers-fast`

Starting the app (not necessary)
`docker run -d -p 3001:3000 node-api-cache-layers-{slow|fast}`

### NextJS App (bigger project)

Change to the next-app project directory
`cd packages/next-app`

Build the non-optimized dockerfile:
`docker build . -f Dockerfile.slow -t next-app-cache-layers-slow`

Build the optimized dockerfile
`docker build . -f Dockerfile.fast -t next-app-cache-layers-fast`

Starting the app (not necessary)
`docker run -d -p 3000:3000 next-app-cache-layers-{slow|fast}`