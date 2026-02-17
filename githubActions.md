# Github Actions

## Table of Content

- [Github Actions](#github-actions)
  - [Table of Content](#table-of-content)
  - [Anatomy of a github action](#anatomy-of-a-github-action)
  - [Possible actions](#possible-actions)
  - [Debugging](#debugging)
  - [Artifacts](#artifacts)

## Anatomy of a github action

We have the name, the inputs if we need them, we specify when it should run them and finally all the steps to take.

Variables can be uploaded in git config. You can add dependencies between steps.

```yml
name: Build and deploy the game

env:
  PROJECT_NAME: BlackJack
  CI: "true"

on:
  push:
    tags: ["v*"]

jobs:
  build:
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            rid: linux-x64
            ext: ""
            folder: linux
            configuration: Release
          - os: windows-latest
            rid: win-x64
            ext: ".exe"
            folder: windows
            configuration: Release
          - os: macos-latest
            rid: osx-x64
            ext: ""
            folder: macos
            configuration: Release
            
    runs-on: ubuntu-latest
    permissions:
      contents: write # Autorise l'action à créer une release et à uploader des fichiers

    # Checkout code
    steps:
    # Préparation
      - name: Checkout Repository
        uses: actions/checkout@v4

      # Install the .NET Core workload
      - name: Install .NET Core
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: 10.0.x

      # Windows (64-bit)
      - name: Publish for ${{ matrix.rid }}
        run: dotnet publish --configuration ${{ matrix.configuration }} --runtime ${{ matrix.rid }} --self-contained true -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true -p UseAppHost=true
        # --self-contained : inclut le runtime .NET pour que le jeu fonctionne partout
        # /p:PublishSingleFile=true : crée un seul fichier .exe, c'est plus propre !
```

```yml
name: "Run tests"
description: "A composite action to run all our tests and check that we don't have problems in our git."

inputs:
  dotnet-version:
    description: "The dotnet version to use to run our tests"
    required: true

runs:
  using: "composite"

  steps:
    - name: Checkout repository
      uses: actions/checkout@v4
    
    - name: Install .NET Core
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ inputs.dotnet-version }}

    - name: Restore dependencies
      run: dotnet restore
      shell: bash

    - name: Build
      run: dotnet build --no-restore
      shell: bash

    - name: Run tests
      run: dotnet test --no-build --verbosity normal
      shell: bash

    - id: run-script
      run: echo "result=Success" >> $GITHUB_OUTPUT
      shell: bash

```

## Possible actions

Github actions can have three different types of actions, container actions (runs on events hosted by github), javascript actions and composite actions. Runners can be hosted by user.

## Debugging

In order to do debugging, add a step like the one under:

```yml
- name: show evt trigger
  run: echo "triggered by ${{github.event_name}}"
```

## Artifacts

Can upload code for public workloads. You can pass artifacts between steps in order to pass folders or others.
