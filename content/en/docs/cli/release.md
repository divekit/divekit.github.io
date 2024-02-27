---
title: "Release"
linkTitle: "Release"
weight: 3
description: >
    After testing the CLI, the next step is to release it. 
    This page describes how the CLI is released and how GoReleaser is used to automate the release process.
---

### What is GoReleaser?

GoReleaser is a tool designed to streamline the process of releasing Go applications. It automates various
tasks associated with the release process, such as compiling binaries, creating release notes, updating
version information, and packaging the application for multiple platforms. GoReleaser aims to simplify the often
complex and manual steps involved in releasing software written in the Go programming language. By using GoReleaser,
developers can save time and ensure a consistent and reliable release process for their Go projects
<a href="https://goreleaser.com/" target="_blank">[1]</a>.

### How is GoReleaser used in this project?

GoReleaser is used to release the CLI binary with the ARS- and Repo-Editor-repository for windows,
but can be extended to other platforms as well. Additionally, the release can be published to any package
manager, such as Scoop, Chocolatey, Homebrew, etc. Currently, it is released to Scoop only, but it is possible to
add more package managers if needed.

To achieve this, GoReleaser provides a simple configuration file that can be used to
configure the release process. The configuration file for this project is located in the root directory and is named
`.goreleaser.yaml`.

The following code snippet shows how the repositories gets included in the release:

```yaml
before:
  hooks:
    - powershell "if (test-path "./repositories") { remove-item  -path "./repositories" -recurse -force }"
    - powershell new-item  -path "./repositories" -type directory
    - powershell ./scripts/setupRepositories.ps1 -destination $PWD/repositories

archives:
  - name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"
    format: zip
    files:
      - src: ./repositories/divekit-automated-repo-setup
        dst: divekit-automated-repo-setup
      - src: ./repositories/divekit-repo-editor
        dst: divekit-repo-editor
```

The `before` section contains a list of hooks that are executed before goReleaser starts the release process.
In this case, the `setupRepositories.ps1` script is executed to clone and setup the ARS- and Repo-Editor-repository
in the `repositories` directory.

The `archives` section defines how the release is packaged and which files are added.
In this case, the CLI binary, ARS- and Repo-Editor-repository are added to the release as zip archives.

The next snippet shows how the release is published to Scoop:

```yaml
scoops:
  - name: divekit
    url_template: "https://github.com/divekit/divekit-cli/releases/download/{{ .Tag }}/{{ .ArtifactName }}"
    homepage: "https://divekit.github.io"
    description: ""
    license: MIT
    skip_upload: true
    depends: [ "umlet", "oraclejre8", "nodejs" ]
```

The `scoops` section contains a list of Scoop properties that defines how the installation with Scoop is handled.
At the end of the release process, a json manifest with these properties plus some manually added properties
gets generated and pushed to the divekit-cli repository. Most properties are self-explanatory,
but some are worth mentioning:

* The `skip_upload` property is set to true to avoid pushing the generated file right away, because some
  properties must be added manually and then pushed. The reason for that is that GoReleaser does not support
  all scoop properties at the moment, such as an environment variable.

* The `depends` property defines which dependencies are required to run the CLI and installs them automatically
  if they are not already installed.

### Why was Scoop chosen for the release of this project?

Scoop is a command-line installer for Windows that is used to install applications from the command line.
It is similar to other package managers, such as Homebrew, but is designed specifically for Windows.

Compared to other Windows package managers such as Chocolatey, Scoop also provides a wide range of apps.
However, it stands out for its simplicity in releasing new versions of applications, because Scoop just reads plain
JSON manifests that describe how to install an application
<a href="https://github.com/ScoopInstaller/Scoop/wiki/Chocolatey-and-Winget-Comparison" target="_blank">[3]</a>.
Additionally, the release process is much faster, because a manual review process is not involved,
which can take several days when using chocolatey.

### What steps are taken to release a new version?

In order to release a new version a workflow file for GitHub is required, which `release_and_test.yml` fulfills.
This gets executed in the CI/CD pipeline, when a commit gets pushed or pulled into the main branch:

1. Cache or restore previously installed dependencies to expedite the installation process.

2. Initialize the project with the `setup.ps1` script. This script installs all necessary dependencies
   and initializes the ARS-, Repo-Editor- and Test-Origin-Repository.

3. Run the tests and if the tests fail, the pipeline gets aborted.

4. Determine the version number automatically based on the commit history of the repository.
   The advantage of this approach is that the version number gets incremented automatically, which reduces the
   risk of human error <a href="https://github.com/PaulHatch/semantic-version#git-based-semantic-versioning"
   target="_blank">[2]</a>.

5. Create a version tag with the determined version number and push it to the repository.

6. Attach the tag to GoReleaser and release the CLI binary with the ARS- and Repo-Editor-repository for windows.

7. Manually add the missing properties to the generated Scoop manifest and push it to the repository.

### References

[1]
<a href="https://goreleaser.com/" target="_blank">"GoReleaser"</a>
(accessed Jan. 29, 2024).

[2]
<a href="https://github.com/PaulHatch/semantic-version#git-based-semantic-versioning" target="_blank">
"PaulHatch/semantic-version"</a>
(accessed Jan. 29, 2024).

[3]
<a href="https://github.com/ScoopInstaller/Scoop/wiki/Chocolatey-and-Winget-Comparison" target="_blank">"Chocolatey and
Winget Comparison"</a>
(accessed Jan. 29, 2024).
