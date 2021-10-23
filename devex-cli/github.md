---
description: DevEx sub-command for GiLab
---

# GitHub

The GitHub tools are offered by the `devex github` \(or `devex gh` for short\) command.
This command offers a set of common options that apply to all sub-commands.
The available sub-commands are:

* `clone` \(`cl` for short\): Clone a complete GitHub organization.

Before we start with real usage commands a few distinctions need to be made.

* Interacting with github.com or with a hosted GitHub installation.
* Interacting with public or private organizations.
* Using `SSH` or `HTTPS` protocols to interact with git repositories.

By default the tools targets github.com, but setting the variable `GITHUB_URL` on the environment will make it target any arbitrary GitHub installation.
The API offered by github.com offers the latest released versions of the endpoints.
While hosted installations offer arbitrary versions of those endpoints.
The tool is built against the latest available endpoints.

Interacting with public or private organizations is not the same.
Public organizations \(and their public repositories\) can be discovered and cloned without needing a personal access token.
Interacting with private organizations always needs a personal access token.
The personal access token must be set on the environment using the variable `GITHUB_TOKEN`.
The token needs at least the `repo` scope.
See GitHub's [Creating a personal access token](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token) for an overview of all available token scopes.

## Clone

GitHub offers the ability to create organizations of repositories and then leverage those organizations to manage multiple repositories at once.
Things like GitHub Actions Secrets, or user membership can be defined at the organization level and then inherited by all the underlying repositories.

It's handy, and sometimes needed, to clone the organizations of repositories preserving the organization structure.
That is what this tool does.

The command takes two parameters, the organization to be cloned and the local path where to clone.
The path parameter is optional and if omitted defaults to the current directory.

```text
devex github clone [-hrvVx] [--debug] [--trace] [-c=<cloneProtocol>] ORGANIZATION PATH
```

Besides the common `devex` command line options, the `github clone` command offers the following options.

* `-r`, `--recurse-submodules`: Initialize repository submodules. If repositories are already cloned try and initialize sub-modules anyway.
* `-c`, `--clone-protocol=<cloneProtocol>`: Chose the transport protocol used clone the repository repositories. Valid values: `SSH`, `HTTPS`. Default is `SSH`.

#### Cloning protocols

To clone via `SSH` \(option: `--clone-protocol=ssh`, default\) the host where the tool runs needs to be configured appropriately with a private key because the tool won't ask for a password.
To clone via `HTTPS` \(option: `--clone-protocol=https`\) the host does not need any setup, and the personal access token will be used as the password for cloning private repositories.

There are three requirements for cloning via `SSH` that apply to both public and private organizations:

1. A known hosts file containing and entry for the GitHub server must exist in the default location \(`${HOME}/.ssh/known_hosts`\).
The command `ssh -T -o 'HostKeyAlgorithms ssh-rsa' git@github.com` can be used populate the `known_hosts` file.
The entry on the file looks something like this: `github.com ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==`
2. A private key must exist in the default location \(`${HOME}/.ssh/id_rsa`\) or in a different location as long there is an entry for the GitHub server in the ssh client configuration \(`${HOME}/.ssh/config`\) that points to the correct key.
See [Connecting to GitHub with SSH](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh) for more information on how to set this up.

### Cloning a public organization from github.com

Cloning public organizations form github.com is the simplest `github clone` command.
The following command clones the organization [devex-cli-example](https://github.com/devex-cli-example) from github.com.

```text
devex github clone devex-cli-example
```

To have the tool also initialize the git submodules of each repository add `--recurse-submodules`.
This can be done in the initial command, or in a second run of the command.
Running the command multiple times with the same parameters either updates the cloned organization with new sub-organizations and repositories, initializes and updates git submodules, or does nothing.

```text
devex github clone --recurse-submodules devex-cli-example
```

The previous commands clone the organization to the current directory.
The local path where the organization is cloned can be customized by adding that path as the last argument.
The following command will clone the organization under the directory `${HOME}/github.com`, creating that directory if it does not exist.

```text
devex github clone devex-cli-example ~/github.com
```

After executing the previous command a directory structure similar to the following should have been created.

```text
tree -d -L 2 ~/github.com/devex-cli-example
/Users/miguel/github.com/devex-cli-example
├── a-public-repository
└── another-public-repository

2 directories
```

Finally, when cloning public organizations from github.com the protocol used for cloning can be changed with a single option.

```text
devex github clone --clone-protocol=https devex-cli-example
```

### Cloning private organizations from github.com

To clone private organizations a token needs to be provided.

```text
GITHUB_TOKEN="..." devex github clone some-private-organization
```

### Cloning from hosted GitHub installations

To clone from a hosted GitHub installation the URL for that installation needs to be provided.

```text
GITHUB_URL="https://github.acme.com" \
GITHUB_TOKEN="..." devex github clone some-private-organization
```
