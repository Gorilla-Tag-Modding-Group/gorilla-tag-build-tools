# Gorilla Tag Build Tools

A collection of tools (mostly composite GitHub Actions) for building Gorilla Tag mods.

These files are intended to work with mods created with the [Gorilla Tag Mod Template](https://github.com/graicc/gorillatagmodtemplate)

## Actions

- `actions/working-dir`: This finds the directory where the `.csproj` file is located, which is necesssary to determine whhere to put files. It outputs this directory as `dir`.

- `actions/setup`: This sets up the build enviroment by downloading the [Stripped Libs](https://github.com/Gorilla-Tag-Modding-Group/BeatStripper) and other common dependencies (e.g. BepInEx).
Requries `working-dir`, which is typically obtained from `actions/working-dir`.

- `actions/deps/[USER]/[REPO]`: These actions set up dependencies that a mod may need. `output-dir` is required, and is typically `[working-dir]\Libs`.

- `actions/build`: This action builds the mod by running [`MakeRelease.ps1`](https://github.com/Graicc/GorillaTagModTemplate#template-information).
Requries `working-dir`, which is typically obtained from `actions/working-dir`.

- `actions/upload`: This action uploads a build to GitHub or a Discord channel.
If a Discord webhook is supplied as `webhook-url`, the mod will be uploaded there, otherwise it will be uploaded as a [build artifact](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts#about-workflow-artifacts).

- `actions/release`: This action creates a GitHub release with the specified `tag` from a template file `template`. Requires `token`, and optionally takes in `human-name` to be used as the repo name.

- `actions/publish`: This action makes a Discord post based on a GitHub release. It must be ran in an action triggered by a release publish.
[Build enviroments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#wait-timer) can be used to delay the post to allow [Monke Mod Info](https://github.com/DeadlyKitten/MonkeModInfo) to update.
[MarkdownDiscordConverter](https://github.com/Graicc/MarkdownDiscordConverter) is used to format the release message for Disord.
Requires `webhook-url` to contain the Discord webhook.

## Usage

An example build script is proided below. Since you may want to build the mod under many conditions (such as when a commit is pushed or a release is made), it may be wise to extract the building to its own composite action, such as the following.

```yml
name: Build
description: Builds mod

runs:
  using: "composite"
  steps:
  # Get working directory for later
  - name: Get working directory
    id: wd
    uses: Gorilla-Tag-Modding-Group/gorilla-tag-build-tools/actions/working-dir@main

  # Set up build enviroment
  - uses: Gorilla-Tag-Modding-Group/gorilla-tag-build-tools/actions/setup@main
    with:
      working-dir: ${{ steps.wd.outputs.dir }}

  # Set up dependencies
  - uses: Gorilla-Tag-Modding-Group/gorilla-tag-build-tools/actions/deps/legoandmars/Utilla@main
    with:
      output-dir: ${{ steps.wd.outputs.dir }}\Libs
  - uses: Gorilla-Tag-Modding-Group/gorilla-tag-build-tools/actions/deps/ToniMacaroni/ComputerInterface
    with:
      output-dir: ${{ steps.wd.outputs.dir }}\Libs

  # Compile the mod to a release zip (using MakeRelease.ps1)
  - uses: Gorilla-Tag-Modding-Group/gorilla-tag-build-tools/actions/build@main
    with:
      working-dir: ${{ steps.wd.outputs.dir }}

  # Upload the mod as a build asset
  - uses: Gorilla-Tag-Modding-Group/gorilla-tag-build-tools/actions/upload@main
```

More examples can be found in the [examples](/examples/) folder.
