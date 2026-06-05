# GitHub Authentication on HPC

How to authenticate to GitHub and `git push` from a TACC compute node, where password authentication fails and the graphical password prompt does not work.

**Guide:** [View the tutorial](https://morehouse-supercomputing.github.io/github-on-hpc/)

## What's covered

- Why `git push` fails on HPC ("password authentication is not supported" + `gnome-ssh-askpass cannot open display`)
- **Method 1:** Personal Access Token over HTTPS (recommended for TACC), including saving it so you never retype it
- **Method 2:** SSH keys (no expiry), with the port-443 workaround for the TACC firewall
- Troubleshooting common errors

## Repo contents

| File | Description |
|------|-------------|
| [index.md](index.md) | The guide (hosted as a GitHub Pages site) |

## Related guides

- [MSCF Getting Started](https://morehouse-supercomputing.github.io/mscf-getting-started/) — account setup, MFA, first job
- [Launching Jupyter on HPC](https://morehouse-supercomputing.github.io/jupyter-on-hpc/) — SSH + idev and Analysis Portal
- [Launching Jupyter via Tapis](https://morehouse-supercomputing.github.io/jupyter-on-tapis/) — Jupyter through the Morehouse Tapis web interface

## Hosted site

This repo is configured for GitHub Pages. The guide is published at:

`https://morehouse-supercomputing.github.io/github-on-hpc/`

To enable: go to **Settings > Pages** and set the source to the `main` branch.

## Maintainer

Ashley Scruse, Deputy Director, Morehouse Supercomputing Facility.
