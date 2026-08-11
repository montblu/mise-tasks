# mise-tasks

Global [mise](https://mise.jdx.dev/) tasks shared across all projects.

## Overview

This repository contains reusable mise tasks intended to be registered globally, making them available in every project without having to repeat them per-repository.

Tasks are shell scripts that follow the [mise task](https://mise.jdx.dev/tasks/) conventions — each file is executable, carries a shebang line, and uses `#MISE` comments for metadata.

## Tasks

### `login`

| Task           | Description                                                                                                                               |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `login:aws`    | Login to AWS via SSO (`aws sso login`). Skips login if a session is already active. Requires `AWS_SSO_PROFILE` to be set.                 |
| `login:az`     | Login to Azure (`az login`). Skips login if a valid token already exists for the configured tenant. Requires `AZURE_TENANT_ID` to be set. |
| `login:gcloud` | Login to GCP using Application Default Credentials (`gcloud auth application-default login`). Skips login if ADC are already valid.       |

### `k`

| Task             | Description                                                                      |
| ---------------- | -------------------------------------------------------------------------------- |
| `k:pod-req-cpu`  | Show resource capacity for all pods ordered by CPU request (`kube-capacity`).    |
| `k:pod-req-mem`  | Show resource capacity for all pods ordered by memory request (`kube-capacity`). |
| `k:pod-util-cpu` | Show pod utilization for all pods ordered by CPU usage (`kube-capacity`).        |
| `k:pod-util-mem` | Show pod utilization for all pods ordered by memory usage (`kube-capacity`).     |

### `tf`

| Task                          | Description                                                                                                                                             |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tf:all-sites-on-env`         | Selects the given `<env>` environment, runs `terrabutler tf` there and switches back to the original environment afterwards. Same `[sites]` / `-a` / `-k` options as below. |
| `tf:sites-on-all-envs`        | Runs `terrabutler tf` for every environment in `environments.permanent`. Defaults to all sites in `sites.ordered`; pass one or more `[sites]` to narrow it, `-a/--action` to pick the action (default: `apply`) and `-k/--keep-going` to continue after a failure. |
| `tf:all-sites-on-current-env` | Runs `terrabutler tf` across all sites in the currently selected environment. Same options as above. Used internally by `tf:all-sites-on-env`.          |
| `tf:summarize`                | Runs a Terraform plan via `terrabutler` for the given site and summarises the output with `tf-summarize`. Accepts a `<site>` argument (default: `k8s`). |

The `tf` tasks read the site and environment lists from the consuming project's `configs/settings.yml` (`general.organization`, `sites.ordered`, `environments.permanent`) and skip any site without a matching `configs/variables/<org>-<env>-<site>.tfvars` file.

## Setup

Point mise at this repository as a task source by adding it to your mise configuration:

```toml
[task_config]
includes = ["git::https://github.com/montblu/mise-tasks.git//tasks?ref=main"]
```

After that, the tasks are available:

```sh
mise login:aws
mise login:az
mise login:gcloud
mise tf:all-sites-on-env staging              # all sites on staging, apply
mise tf:all-sites-on-env staging k8s -a plan  # single site on staging, plan
mise tf:all-sites-on-current-env -a plan      # all sites on the current env, plan
mise tf:sites-on-all-envs                     # all sites, apply
mise tf:sites-on-all-envs k8s                 # single site
mise tf:sites-on-all-envs k8s helm -a plan    # several sites, plan
mise tf:summarize          # defaults to site=k8s
mise tf:summarize helm
mise k:pod-req-cpu
mise k:pod-req-mem
mise k:pod-util-cpu
mise k:pod-util-mem
```

## Environment Variables

| Variable          | Required by | Description                             |
| ----------------- | ----------- | --------------------------------------- |
| `AWS_SSO_PROFILE` | `login:aws` | AWS CLI profile name used for SSO login |
| `AZURE_TENANT_ID` | `login:az`  | Azure Active Directory tenant ID        |

These can be set in your shell profile or in the global mise environment (`~/.config/mise/config.toml` under `[env]`).
