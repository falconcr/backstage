# Backstage
Backstage provides tooling to build Docker images, but can be deployed with or without Docker on many different infrastructures. The best way to deploy Backstage is in the same way you deploy other software at your organization.

At Spotify, they deploy software generally by:

- Building a Docker image
- Storing the Docker image on a container registry
- Referencing the image in a Kubernetes Deployment YAML
- Applying that Deployment to a Kubernetes cluster

# Prerequisites
This guide assumes a basic understanding of working on a Linux based operating system and have some experience with the terminal, specifically, these commands: npm, yarn.

- Access to a Unix-based operating system, such as Linux, macOS or Windows Subsystem for Linux
- A GNU-like build environment available at the command line. For example, on Debian/Ubuntu you will want to have the make and build-essential packages installed. On macOS, you will want to have run xcode-select --install to get the XCode command line build tooling in place.
- curl or wget installed
- Node.js Active LTS Release installed using one of these methods:
    - Using nvm (recommended)
    - Installing nvm
    - Install and change Node version with nvm

- yarn 
- docker installation
- git installation

# Demo

## Create your Backstage App

To install the Backstage Standalone app, we will make use of npx. npx is a tool that comes preinstalled with Node.js and lets you run commands straight from npm or other registries. Before we jump in to running the command, let's chat about what it does.

```
npx @backstage/create-app@latest
```

Inside that directory, it will generate all the files and folder structure needed for you to run your app.

General folder structure
Below is a simplified layout of the files and folders generated when creating an app.

```
app
├── app-config.yaml
├── catalog-info.yaml
├── package.json
└── packages
    ├── app
    └── backend
```

- app-config.yaml: Main configuration file for the app. See Configuration for more information.
- catalog-info.yaml: Catalog Entities descriptors. See Descriptor Format of Catalog Entities to get started.
- package.json: Root package.json for the project. Note: Be sure that you don't add any npm dependencies here as they probably should be installed in the intended workspace rather than in the root.
- packages/: Lerna leaf packages or "workspaces". Everything here is going to be a separate package, managed by lerna.
- packages/app/: An fully functioning Backstage frontend app, that acts as a good starting point for you to get to know Backstage.
- packages/backend/: We include a backend that helps power features such as Authentication, Software Catalog, Software Templates and TechDocs amongst other things.

```
cd my-backstage-app # your app name
yarn dev
```

## Dockerize

```
yarn install --immutable

# tsc outputs type definitions to dist-types/ in the repo root, which are then consumed by the build
yarn tsc

# Build the backend, which bundles it all up into the packages/backend/dist folder.
# The configuration files here should match the one you use inside the Dockerfile below.
yarn build:backend --config ../../app-config.yaml --config ../../app-config.production.yaml
```

Once the host build is complete, we are ready to build our image

The Dockerfile is located at packages/backend/Dockerfile, but needs to be executed with the root of the repo as the build context, in order to get access to the root yarn.lock and package.json, along with any other files that might be needed, such as .npmrc.

```
docker image build . -f packages/backend/Dockerfile --tag glgelopfalcon/backstage
```

```
docker compose up
```

Check backstage running `http://localhost:7007/`
