---
description: DevEx sub-command for GiLab
---

# GitLab

The GitLab tools are offered by the `devex gitlab` \(or `devex gl` for short\) command. This command offers a set of common options that apply to all sub-commands. The available sub-commands are:

* `clone` \(`cl` for short\): Clone a complete GitLab group preserving the path hierarchy of the group.

Before we start with real usage commands a few distinctions need to be made.

* Interacting with gitlab.com or with a hosted GitLab installation.
* Interacting with public or private groups and projects.
* Using `SSH` or `HTTPS` protocols to interact with git repositories.

By default the tools targets gitlab.com, but setting the variable `GITLAB_URL` on the environment will make it target any arbitrary GitLab installation. The API offered by gitlab.com offers the latest released versions of the endpoints. While hosted installations offer arbitrary versions of those endpoints. The tool is built against the latest available endpoints, but for certain operations the API version is checked and fallback implementations for older versions can be offered. When the tool tries to cal an API endpoint that isn't available the endpoint will return a `404 Not Found` error.

Interacting with public or private groups is not the same. Public groups \(and their public projects\) can be discovered and cloned without needing a private token. Interacting with private groups always needs a private token. The private token must be set on the environment using the variable `GITLAB_TOKEN`. The token needs at least the `read_api` scope. See GitLab's [Limiting scopes of a personal access token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#limiting-scopes-of-a-personal-access-token) for more details on token scopes.

## Clone

GitLab offers the ability to create groups of repositories and then leverage those groups to manage multiple repositories at once. Things like CI/CD, or user membership can be defined at the group level and then inherited by all the underlying repositories. Furthermore, it's also possible to create relationships between repositories simply by leveraging the group structure. For example, one can include git sub-modules and reference them by their relative path.

It's handy, and sometimes needed, to clone the groups of repositories preserving the group structure. That is what this tool does.

The command takes two parameters, the group to be cloned and the local path where to clone. The path parameter is option and f omitted defaults to the current directory.

```text
devex gitlab clone [-hrvVx] [--debug] [--trace] [-c=<cloneProtocol>] [-m=<searchMode>] GROUP PATH
```

Besides the common `devex` command line options, the `gitlab clone` command offers the following options.

* `-r`, `--recurse-submodules`: Initialize project submodules. If projects are already cloned try and initialize sub-modules anyway.
* `-c`, `--clone-protocol=<cloneProtocol>`: Chose the transport protocol used clone the project repositories. Valid values: `SSH`, `HTTPS`. Default is `SSH`.
* `-m`, `--search-mode=<searchMode>`: Chose how the group is searched for. Groups can be searched by name or full path. Valid values: `NAME`, `FULL_PATH`, `ID`. Default is `NAME`.

#### Cloning protocols

To clone via `SSH` \(option: `--clone-protocol=ssh`, default\) the host where the tool runs needs to be configured appropriately with a private key because the tool won't ask for a password. To clone via `HTTPS` \(option: `--clone-protocol=https`\) the host does not need any setup, and the private token will be used as the password for cloning private projects.

There are two requirements for cloning via `SSH` that apply to both public and private groups.

1. A known hosts file containing and entry for the GitLab server must exist in the default

   location \(`${HOME}/.ssh/known_hosts`\). The command `ssh -T -o 'HostKeyAlgorithms ecdsa-sha2-nistp256' git@gitlab.com` can be used populate the `known_hosts` file. The entry on the file looks something like this: `gitlab.com,172.65.251.78 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFSMqzJeV9rUzU4kWitGjeR4PWSa29SPqJ1fVkhtj3Hw9xjLVXVYrU9QlYWrOLXBpQ6KWjbjTDTdDkoohFzgbEY=`

2. A private key must exist in the default location \(`${HOME}/.ssh/id_rsa`\) or in a different location as long there is an entry for the GitLab server in the ssh client configuration \(`${HOME}/.ssh/config`\) that points to the correct key.

   See [GitLab documentation on configuring SSH access](https://docs.gitlab.com/ee/ssh/) for more information on how to set this up.

### Cloning a public group from gitlab.com

Cloning public groups form gitlab.com is the simplest `gitlab clone` command. The following command clones the group [open-source-dexex](https://gitlab.com/open-source-devex) from gitlab.com.

```text
devex gitlab clone open-source-devex
```

To have the tool also initialize the git submodules of each repository add `--recurse-submodules`. This can be done in the initial command, or in a second run of the command. Running the command multiple times with the same parameters either updates the cloned group with new sub-groups and projects, initializes and updates git submodules, or does nothing.

```text
devex gitlab clone --recurse-submodules open-source-devex
```

The previous commands clone the group to the current directory. The local path where the group is cloned can be customized by adding that path as the last argument. The following command will clone the group under the directory `${HOME}/gitlab.com`, creating that directory if it does not exist.

```text
devex gitlab clone open-source-devex ~/gitlab.com
```

After executing the previous command a directory structure similar to the following should have been created.

```text
tree -d -L 2 ~/gitlab.com/open-source-devex
/Users/miguel/gitlab.com/open-source-devex
├── containers
│ ├── application
│ ├── bastion
│ ├── build
│ ├── build-java
│ ├── build-node
│ ├── build-packer
│ ├── build-terraform
│ ├── ecr-container-scan-kickstarter
│ ├── inspector-kickstarter
│ ├── java
│ ├── keybase
│ ├── maintenance-page
│ └── spring-boot-admin
├── gitlab-ci-config
│ └── terraform
├── install-me
│ ├── git-flow
│ └── webapp
├── serverless
│ └── aws
├── terraform-modules
│ ├── 0-cicd
│ ├── aws
│ ├── devops
│ ├── gitlab
│ ├── kubernetes
│ └── utils
├── terraformed-systems
│ └── example
└── toolbox
    ├── aws
    ├── backups
    ├── crypto
    ├── db
    ├── docker
    ├── fs
    ├── git
    └── users

39 directories
```

To clone a group that has a name with spaces, wrap the name in quotes.

```text
devex gitlab clone "Healthcare Research Analysis"
```

The tools support other ways of specifying the group to be cloned. The group can be selected by its GitLab id, or via full path. Both options are very useful when cloning GitLab sub-groups.

```text
devex gitlab clone --search-mode=id 2150497
```

```text
devex gitlab clone --search-mode=full_path open-source-devex/terraform-modules/aws
```

Finally, when cloning public groups from gitlab.com the protocol used for cloning can be changed with a single option.

```text
devex gitlab clone --clone-protocol=https open-source-devex
```

### Cloning private groups from gitlab.com

To clone private groups a token needs to be provided.

```text
GITLAB_TOKEN="..." devex gitlab clone "some private group"
```

### Cloning from hosted GitLab installations

To clone from a hosted GitLab installation the URL for that installation needs to be provided.

```text
GITLAB_URL="https://gitlab.acme.com" \
GITLAB_TOKEN="..." devex gitlab clone "some private group"
```
