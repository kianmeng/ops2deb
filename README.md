[![upciti](https://circleci.com/gh/upciti/debops.svg?style=svg)](https://circleci.com/gh/upciti/debops)
[![codecov](https://codecov.io/gh/upciti/debops/branch/main/graph/badge.svg)](https://codecov.io/gh/upciti/debops)
[![MIT license](https://img.shields.io/badge/License-MIT-blue.svg)](https://lbesson.mit-license.org/)
[![Generic badge](https://img.shields.io/badge/type_checked-mypy-informational.svg)](https://mypy.readthedocs.io/en/stable/introduction.html)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)
[![PyPI version shields.io](https://img.shields.io/pypi/v/debops.svg)](https://pypi.python.org/pypi/debops/)
[![Downloads](https://static.pepy.tech/personalized-badge/debops?period=total&units=international_system&left_color=blue&right_color=green&left_text=Downloads)](https://pepy.tech/project/debops)

# debops

Are you tired of checking if your favorite devops tools are up-to-date? Are you using a debian based GNU/Linux distribution? 
Debops is designed to generate Debian packages for common devops tools such as kubectl, kustomize, helm, ...,
but it could be used to package any statically linked application. In short, it consumes a configuration file and outputs `.deb` packages.

## Configuration file

Written in YAML and composed of a list of package blueprints. A blueprint is defined by the following:


| Field         | Meaning                                                                                        | Default      |
| ------------- | ---------------------------------------------------------------------------------------------- | ------------ |
| `name`        | Component name, e.g. `kustomize`                                                               |              | 
| `version`     | Application release to package                                                                 |              |
| `arch`        | Package architecture                                                                           | `amd64`      |
| `revision`    | Package revistion                                                                              | `1`          |
| `summary`     | Package short description                                                                      |              |
| `description` | Package full description                                                                       | `Null`       |
| `fetch`       | A binary to download, and a `sha256` checksum. `tar.gz` archives are extracted automatically   |              |
| `script`      | A list of build instructions templated with jinja2 and intepreted with the default `shell`     |              |

Example: 

```yaml
- name: kubectl
  version: 1.20.1
  summary: Command line client for controlling a Kubernetes cluster
  description: |
    kubectl is a command line client for running commands against Kubernetes clusters.
  fetch:
    url: https://storage.googleapis.com/kubernetes-release/release/v{{version}}/bin/linux/amd64/kubectl
    sha256: 3f4b52a8072013e4cd34c9ea07e3c0c4e0350b227e00507fb1ae44a9adbf6785
  script:
    - mv kubectl {{src}}/usr/bin/
```

## Dependencies

* Python >= 3.8
* To build debian packages with `debops build` you need the following packages on your host:

```shell
sudo apt install fakeroot debhelper
```

## Usage example

Install `debops` in a virtualenv or with [pipx](https://github.com/pipxproject/pipx)

```shell
pipx install debops
```

Then, in a test directory run:

```shell
curl https://raw.githubusercontent.com/upciti/debops/main/debops.yml
debops generate
debops build
```

To check for new releases run:

```shell
debops update
```

`debops` uses temp directories to cache downloaded binaries and to run build instructions:

```shell
tree /tmp/debops_*
```

The cache can be flushed with:
```shell
debops purge
```

## Development

You will need [poetry](https://python-poetry.org/)

```shell
poetry install
poetry run task check
```

## Important notes

`debops` **DOES NOT** sandbox build instructions so if you do something like:

```shell
script:
- rm -rf ~/*
```

You will loose your files... To make sure that you won't mess with your system, run it within a container.
