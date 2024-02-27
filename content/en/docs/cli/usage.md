---
title: "Usage"
linkTitle: "Usage"
weight: 1
description: >
  This page describes the usage of the CLI.
---

{{% pageinfo %}}
The following documentation is more a concept than a description of the finished application.
{{% /pageinfo %}}

## Getting started

### Prerequisites

- `npm`
- `python`
- `pip`

### usage

```bash
npm i @divekit/cli -g # install

mkdir st-2042 # create empty project directory
cd st-2042

divekit init -g 152 -c 14 # initializes in GitLab Dir with groupId 152 and with config base from project in GitLab dir 14
```

### file structure

```yml
st-2042
├── .divekit # all configs, tools etc. needed for this project and the divekit-cli
|   ├── config
|   |    ├── automated-repo-setup
|   |    ├── ...
|   |    └── basic.yml # name, milestone data, ...
|   └── tools
|        ├── access-manager
|        └── ...
├── milestone1
|   ├── config.yml # milestone specific domain-config
|   └── origin-repo
|       └── ...
├── milestone2
└── ...
```

#### basic.yml example

```yml
name: ST1-2022
rootStaffGroupId: 42
rootStudentGroupId: 43
overviewGroupId: 44
milestones:
  m1:
    groupId: 54
    start: 20.08.2023
    end: 04.09.2023
    codeRepoGroupId: 142
    testRepoGroupId: 143
  m2:
    groupId: 67
    start: 07.09.2023
    end: 27.09.2023
    codeRepoGroupId: 152
    testRepoGroupId: 153
  # ...

```

## Functionality

### `divekit init`

Initialize a new divekit project, done once per semester and module. Must be used in an empty directory otherwise it
only checks and installs existing dependencies needed for *tools*. An example for a generated file structure can be seen
above.

#### Options

* `-w / --wizard` install via wizard which asks and explains all parameters and gives options if available
* `-g / --groupId` groupId for the corresponding GitLab directory (should be empty as well)
* `-n / --name` set a name for the project (Defaults to name of parent directory)
* `-c / --config <groupId>` copy a previous used config from given project (*Nice to have*: also works on old projects)

#### errors

* exits if node, python or pip are not available

#### Example usage

Creating a new project with the help of the wizard.

```bash
mkdir st1-2023
divekit init --wizard
$ "Please enter the GitLab GroupId where all Staff content will be managed (should be empty):"
$ 150 # <user_input>
$ "Please enter the GitLab GroupId where all student repositories will be published:"
$ 151 # <user_input>
$ "Please enter a name for the project (default: st1-2023)"
$ # <user_input>
$ "Do you want to use a already existing configuration, then please enter a groupId for the corresponding project otherwise a basic config will be initialized"
$ 42 # <user_input>
$ "Divekit initialized successfully"
# Result: Directory .divekit created locally and pushed to GitLab
```

### `divekit milestone`

manage the creation, testing, publication and more for a milestone.

*if not otherwise configured naming, config etc. will be copied from the last milestone*

#### Options

* `-w / --wizard` create the next milestone via wizard which asks and explains all parameters and gives options if
  available (for more information see the example below)
* `-n / --name <string>` set a custom name for the milestone (default to milestone<Id>)
* `-t / --template <groupId>` create the next milestone based of an already existing origin repo
* `-f / --fresh` while creating a new milestone ignore already existing and their configuration
* `-i / --id <number>` specify milestoneId (default start with 1 or last existing milestone +1)
* `-l / --local <number>` create a given number student-repos locally in demo-folder
* `-r / --random <none || all | roundrobin>` configure randomization. None at all, Everything (like normal student repo
  generation) or roundrobin which tries to create uniform distribution of all variations
* `-p / --publish` creates and publishes all repos for students based on origin repo
* `-d / --demo <repoCount>` create a given count of test repositories

#### Example usage

Create the first milestone with the help of the wizard

```bash
divekit milestone --wizard
$ "creating the first milestone of this project"
$ "please enter a number for the first milestone (default: 1)"
$ # <user_input>
$ "Do you want to use a already existing milestone as template?"
$ "Please enter the groupId otherwise skip with Enter"
$ 47 # <user_input>
$ "Milestone created - pls configure the origin repository"
# Result:
# - milestone1 directory created based on template repo
# - updated configs with new milestone and push everything to GitLab
# - also created directories for code/test repositories in GitLab.
```

Create further milestone with the help of the wizard

```bash
divekit milestone --wizard
$ "creating a milestone for st1-2023"
$ "please number for the milestone (default: 2)"
$ # <user_input>
$ "Do you want to use a already existing milestone as template?"
$ "Please enter the groupId otherwise the milestone2 will be based on milestone1"
$ # <user_input>
$ "Milestone created - pls configure the origin repository"
# Result:
# - milestone2 directory created based on milestone1 repo
# - updated configs with new milestone and push everything to GitLab
# - also created directories for code/test repositories for milestone2 in GitLab.
```

### `divekit patch`

Edit the current or given milestone.

#### Options

* `-m / --milestone <numer>` number of milestone you want to edit (default: latest existing milestone)
* `-g / --g <[groupId]>` list of groupIds where to publish edits (default: student repositories of the milestone)
* `-f / --files <[names]>` list of files to edit/create
* `-c / --commit <msg>`: custom commit msg (default: "fix 42")
* `-d / --debug`: sets loglevel to debug

## Installation

Currently, the CLI is only available for Windows. To install the CLI, follow these steps:

1. Open a PowerShell window as user.
2. Allow scripts to be executed:
   ```ps1 
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```
3. Install scoop:
   ```ps1 
   iwr -useb get.scoop.sh | iex
   ```
4. Add buckets with the following commands to find the required apps:
   ```ps1 
   scoop bucket add divekit https://github.com/divekit/divekit-cli
   scoop bucket add extras
   scoop bucket add java
   ```
5. To install Divekit along with its dependencies, use:
   ```ps1 
   scoop install divekit
   ```

### Use Cases and other ideas - which should be considered

- create repos for latecomers, e.g. because they passed the previous milestone after a discussion or forgot to register.
- divide the config structure between technical and domain configuration