# Overview

## Getting started

To install the `devex` command line tool you can either download [a version from GitHub](https://github.com/miguelaferreira/devex-cli/releases) or use Homebrew to install the latest version on either Linux or macOS.

```text
brew install miguelaferreira/tools/devex-cli
```

The source code is available at [https://github.com/miguelaferreira/devex-cli](https://github.com/miguelaferreira/devex-cli).

## Introduction

The `devex` command line tool is actually a collection of tools that automate some of the gruntwork that developers need to do. For example, and this is the first use-case the tool supports, GitLab offers a feature called Groups that helps to manage and organise multiple repositories. When developing on a project that hosts its codebase in GitLab it can be useful to clone all the repositories in the same group while preserving the path hierarchy of the group. The `devex` command line tool offers a sub-command that does just that, in this case `devex gitlab clone`. The equivalent operation on GitHub would be to clone an entire organization, the command for that is `devex github clone`.

As you probably noticed already the tools offered are organised in sub-commands. These tools help in automating tasks performed against APIs \(such as GitLab, GitHub or AWS\) as well as tools \(such as git or terraform\). The tools, and the respective sub-commands, are typically named after the thing they automate against.

## Tools

* [GitLab](gitlab.md)
* [GitHub](github.md)

## Common command line options

All tools support the following command line options.

* `-h`, `--help`: Show this help message and exit.
* `-V`, `--version`: Print version information and exit.
* `-v`, `--verbose`: Print out extra information about what the tool is doing.
* `-x`, `--very-verbose`: Print out even more information about what the tool is doing.
* `--debug`: Sets all loggers to DEBUG level.
* `--trace`: Sets all loggers to TRACE level. ⚠️ WARNING: this setting will leak tokens used for HTTP authentication \(eg. the GitLab token\) to the logs, use with caution. ⚠️

