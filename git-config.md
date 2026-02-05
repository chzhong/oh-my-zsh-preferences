# Configure different git identities

Allow you use different identities for git repositorites based on git remotes or directories. e.g.:

- remotes on github: use your github identity for open-source projects
- remotes on your company github/gitlab: use your stuff identity for your works

You won't have to do per-repo configuration.

## Configure you default identy globally

```
git config --global user.name "Stuff Name"
git config --global user.email "account@company.com"
```

## Create `~/.gitconfig-`_env_ files

For example, create an config for github in `~/.gitconfig-github`:

```ini
[user]
        email = 1****+githuber@users.noreply.github.com
        name = Github Name
        ; signingkey = 1234567890ABCDEF    ; your Github GPG key
```
You can use noreply email provided by github to protect your privacy.

Also, create an config for github in `~/.gitconfig-github`:

```ini
[user]
        email = account@company.com
        name = Stuff Name
        ; signingkey = FEDCBA9876543210 ; your company GPG key                       
```
You can use noreply email provided by github to protect your privacy.


## Use `includeIf` to conditionally inlcude `~/.gitconfig-`_env_ files

```ini
; other global git configs

; conditionally include identity-specified configs based on remotes or direcdtory, overriding global configs
[includeIf "hasconfig:remote.*.url:https://gitlab.company.com/**/*.git"] ; requires Git version >= 2.36
    path = ~/.gitconfig-work

[includeIf "hasconfig:remote.*.url:git@gitlab.company.com:**/*.git"]
    path = ~/.gitconfig-work

[includeIf "gitdir:~/workspace/work/"]
    path = ~/.gitconfig-work

[includeIf "hasconfig:remote.*.url:https://github.com/**/*.git"]
    path = ~/.gitconfig-github

[includeIf "hasconfig:remote.*.url:git@github.com:**/*.git"]
    path = ~/.gitconfig-github

[includeIf "gitdir:~/workspace/github/"]
    path = ~/.gitconfig-github
```

### Test configuration

In your github repo, type:
```bash
git config --show-origin user.name
git config --show-origin user.email
```
to see whether your github identity is used from `~/.gitconfig-github`.

In your work repo, type:
```bash
git config --show-origin user.name
git config --show-origin user.email
```
to see whether your company identity is used from `~/.gitconfig-work` or default config (if you use it as default).

