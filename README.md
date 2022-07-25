# Gorilla Tag Build Tools

A collection of tools (mostly composite GitHub Actions) for building Gorilla Tag mods.

These actions are intended to work with mods created with the [Gorilla Tag Mod Template](https://github.com/graicc/gorillatagmodtemplate).

## Actions

### [working-dir](actions/working-dir/)

This finds the directory where the `.csproj` file is located, which is necesssary to determine whhere to put files.

Outputs:

- `dir`: The directory the project is located in

### [setup](actions/setup)

This sets up the build enviroment by downloading the [Stripped Libs](https://github.com/Gorilla-Tag-Modding-Group/BeatStripper) and other common dependencies (e.g. BepInEx).

Inputs:

- `working-dir`: Working directory, typically the output of [working-dir](actions/working-dir/)

### [deps/[USER]/[REPO]](actions/deps)

These actions set up dependencies that a mod may need.

Inputs:

- `output-dir`: Directory to put files for reference, typically `[working-dir]\Libs`

### [build](actions/build)

This action builds the mod by running [`MakeRelease.ps1`](https://github.com/Graicc/GorillaTagModTemplate#template-information).

Inputs:

- `working-dir`: Working directory, typically the output of [working-dir](actions/working-dir/)

### [upload](actions/upload)

This action uploads a build to GitHub or a Discord channel.

Inputs:

- `webhook-url`: Optional Discord webhook url. If supplied, the build will be uploaded there, otherwise it will be uploaded as a [build artifact](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts#about-workflow-artifacts)
- `file-path`: Optional flie path to build. Defaults to `Build.zip`

### [release](actions/release)

This action creates a GitHub release from a template file. Applies the following subsitutions to the template file:

- `{NAME}` -> Name of the repository
- `{HUMAN_NAME}` -> Human readable name of the repository, from `human-name` input
- `{DESCRIPTION}` -> Description of the repository (i.e. the about tab)
- `{REPO}` -> Full url of the github repository, e.g. <https://github.com/Gorilla-Tag-Modding-Group/gorilla-tag-build-tools>
- `{VERSION}` -> Tag specified for the release

Inputs:

- `token`: Github token, typically `${{ secrets.GITHUB_TOKEN }}`
- `tag`: Tag for the release (e.g. 1.2.3)
- `template`: Optional path to tempalte file. Defaults to `.github/templates/release.md`
- `human-name`: Optional human readable name of the repository. Defaults to `${{ github.event.repository.name }}`
`file-path`: Optional flie path to build. Defaults to `Build.zip`

### [publish](actions/publish)

This action makes a Discord post based on a GitHub release. It must be ran in an action triggered by a release publish.
[Build enviroments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#wait-timer) can be used to delay the post to allow [Monke Mod Info](https://github.com/DeadlyKitten/MonkeModInfo) to update.
[MarkdownDiscordConverter](https://github.com/Graicc/MarkdownDiscordConverter) is used to format the release message for Disord.

Inputs:

- `webhook-url`: Discord webhook url

## Usage

An example build script is proided below. Since you may want to build the mod under many conditions (such as when a commit is pushed or a release is made), it may be wise to extract the building to its own composite action, such as the following:

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
