---
aliases:
  - Git
tags:
  - learning
  - dev/version-control
date: 2026-07-03
---
**Sources**: [Git](https://git-scm.com/), [GUIs](https://git-scm.com/tools/guis)

**Related:** [[Version Control]]

---

## Description

_Git_ is a **free and open source** distributed ``version control system`` designed to handle everything from small to very large projects with speed and efficiency.

Git is lightning fast and has a huge ecosystem of GUIs, hosting services, and command-line tools.

---

## Key commands

### Usage

| Command                                        | Usage                                                                                                                                         |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **git branch -m main**                         | Change the default initial branch from "master" to "main"                                                                                     |
| **git status --short**                         | Shows a short version of the current changes                                                                                                  |
| **git log --oneline --graph --decorate --all** | Show commits history log shortly and visually                                                                                                 |
| **git branch -D** _branch_name_                | Deletes a branch                                                                                                                              |
| **git tag -a v1.0 -m "First version"**         | Adds the tag "v1.0" to the last commit for a better identification                                                                            |
| **git tag**                                    | List all history tags                                                                                                                         |
| **git show v1.0**                              | Shows details about that specific tag                                                                                                         |
| **git tag -d v1.0**                            | Removes a tag from its linked commit without making changes to that commit                                                                    |
| **git checkout** _commit_id_                   | Moves the working directory to a detached branch to explore previous changes and making modifications without modifying any important branch  |

### Global settings

| Command                                         | Usage                                                                                       |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **git config --global init.defaultBranch main** | Change the name of the default branch created when initializing a new repo using _git init_ |
| **git config --global --list**                  | Shows the user name and user email saved                                                    |

---

## Details

### Stage

In technical terms, the _staging area_ is **the middle ground between what you have done to your files (also known as the** _working directory_**) and what you had last committed (the** _HEAD commit_**)**.

As the name implies, the _staging area_ gives you space to prepare (_stage_) the changes that will be reflected on the next _commit_.

![[git_stage.png]]


### SSH Security Settings

![[git_ssh.png]]

#### Generation

Execute the following command and enter a passphrase:

```bash title:bash
ssh-keygen -t ed25519 -C "email@domain.com"
```

Verify the key's creation:

```bash title:bash
ls ~/.shh -la

# Directory: ~\.ssh
# Mode                LastWriteTime         Length Name
# ----                -------------         ------ ----
# -a---     04/07/2026  03:26 p. m.            419   id_ed25519
# -a---     04/07/2026  03:26 p. m.            107   id_ed25519.pub
# -a---     09/12/2025  10:22 p. m.             93   known_host
```


#### Addition to the computer


Verify the ssh agent is running by entering:

```bash title:bash
eval "$(ssh-agent -s)"

# Agent pid 17218
```

```powershell title:powershell
Get-Service -Name ssh-agent

# Status   Name               DisplayName
# ------   ----               -----------
# Stopped  ssh-agent          OpenSSH Authentication Agent
```

If not running, start it with:

```powershell title:powershell
Set-Service -StartupType Manual | Start-Service -Name ssh-agent

Get-Service -Name ssh-agent

# Status   Name               DisplayName
# ------   ----               -----------
# Running  ssh-agent          OpenSSH Authentication Agent
```

Add the key to the ssh agent using the entered passphrase:

```bash title:"bash and powershell"
ssh-add ~/.shh/id_ed25519

# Identity added: ~/.ssh/id_ed25519 (email@domain.com)
```


#### Addition to GitHub

Copy the public key (.pub file) and add it to github.com profile in
$$ Settings > SSH \ and \ GPG \ keys$$

Verify the SSH connection with the following command:

```bash title:"bash and powershell"
ssh -T git@github.com

# Hi user! You've successfully authenticated, but GitHub does not provide shell access.
```


---

## Commands

### Reset

This command **takes you back to a previous commit**. This is useful to see the state inside the working directory as if you had just written those changes.


### Revert

This command **adds the selected previous commit at the top of the commits history as a new one**. This is useful to fix mistakes as a new commit. 


### Fetch

This command retrieves the latest information from the remote repository, updating your remote tracking branch (origin/main). **This allows you to see if new commits or changes have been made by your teammate without merging them into your local branch (main) right away.**

Executing a _pull origin main_ command is the equivalent of executing:
```bash title:bash
git fetch origin main
git merge origin/main
```

To explore the changes it is needed to use the following commands:


#### See full _diff_

```bash title:bash
git diff HEAD origin/main
```

This shows you, line by line, what has changed between your current branch and the remote branch.

If you want to compare it to your specific tracking branch:

```bash title:bash
git diff main origin/main
```


#### See specific file's _diff_

```bash title:bash
git diff HEAD origin/main -- file/path.py
```


#### See new _commits_ and its changes

```bash title:bash
git log HEAD..origin/main -p
```

The _-p_ (patch) option displays the diff for each new commit, not just the message. Without _-p_, you would only see the list of commits.


#### Show only which files have changed (without line-by-line details)

```bash title:bash
git diff --stat HEAD origin/main
```

Useful for getting a quick overview before diving into the details.

---

## Claude Sessions
