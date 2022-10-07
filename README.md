## Overview

The purpose of this repo is to show the time difference between building a docker image that does not take advantage of layer caching vs. the time it takes to build a docker image that does take advantage of layer caching.

## Requirements

- Docker installed

## Running the demo

### Node API (simple project)

Change to the node-api project directory
`cd packages/node-api`

Build the non-optimized dockerfile:
`docker build . -f Dockerfile.slow -t docker-cache-layers-demo-slow`

Build the optimized dockerfile
`docker build . -f Dockerfile.fast -t docker-cache-layers-demo-fast`

### NextJS App (bigger project)