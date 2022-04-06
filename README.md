# Reddit Place Clone Web App

Web application for https://github.com/rdeepak2002/reddit-place-clone-server

## Requirements
- NodeJS (https://nodejs.org/en/)
- Yarn (https://classic.yarnpkg.com/lang/en/docs/install/#mac-stable)

## Get Started

Modify Google authentication client id in App.svelte

```shell
yarn
yarn dev
```

## Build for Production

```shell
yarn
yarn build
```

## Recommended Pre-Commit Git Hooks

Create a file in .git/hooks with the following content:

```shell
#!/bin/sh
yarn build
git add .
```

Make the script executable with the following command:

```shell
sudo chmod 777 .git/hooks/pre-commit
```