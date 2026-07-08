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

| Task           | Description                                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tf:summarize` | Runs a Terraform plan via `terrabutler` for the given site and summarises the output with `tf-summarize`. Accepts a `<site>` argument (default: `k8s`). |

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
