# preprocessing-base

Enduro child workflow base repository. This project is a basic example and a
template for new Enduro child workflow projects.

Despite the `preprocessing-base` name, this repository is also a useful starting
point for projects that implement other workflow types, or projects that run
multiple workflows and Temporal workers from the same binary.

- [Existing repositories](#existing-repositories)
- [Create a new repository](#create-a-new-repository)
- [Repository requirements](#repository-requirements)
- [Configuration](#configuration)
- [Local environment](#local-environment)
- [Makefile](#makefile)

## Existing repositories

Projects based on this template include:

- [preprocessing-sfa](https://github.com/artefactual-sdps/preprocessing-sfa)
- [cva-enduro-workflows](https://github.com/artefactual-sdps/cva-enduro-workflows)

## Create a new repository

To create a new Enduro child workflow project:

- Use this repository as a template:
  - With the link in the top right corner of this page
  - Or selecting it from the create new repository template dropdown
- Replace references to `preprocessing-base` in the code, this will change:
  - The Go module name
  - The default Docker image name
  - The Makefile project name and the location of the installed tools
  - The `appName` in the worker command
  - The gci "prefix" section in the `.golangci.yml` config file
  - The `CHILD_WORKFLOW_PATHS` in the local environment setup below.
- Update this readme file:
  - Change the heading and initial description
  - Remove the first three sections from the list above and the content
  - Update the configuration based on the workflow implementation
  - Update the Enduro `childWorkflows` configuration examples

## Repository requirements

Projects based on this repository are expected to run as child workflow workers
inside the Enduro development environment. Enduro owns the local development
cluster and core services, including Temporal, MySQL and shared infrastructure.
This repository contributes only the child workflow resources needed by its
worker.

The default template includes a persistent volume claim called
`preprocessing-pvc`, mounted at `/home/preprocessing/shared` in the child
workflow worker. Enduro can mount the same volume in its a3m or Archivematica
worker so both workers can share files. This shared-volume development setup is
intended for single-node Kubernetes clusters.

Projects that do not need a shared filesystem, or that implement
non-preprocessing workflow types, should replace these manifests with the
resources required by their own workflow workers.

Check the [Enduro development manual] for the current development environment
setup.

## Configuration

The default preprocessing worker needs to share the filesystem with Enduro's
a3m or Archivematica workers. It must connect to the same Temporal server and be
related to Enduro with the correct namespace, task queue and workflow name.

### Worker configuration

The required configuration for the default preprocessing worker:

```toml
debug = false
verbosity = 0
sharedPath = "/home/preprocessing/shared"

[temporal]
address = "temporal-frontend.enduro-sdps:7233"
namespace = "default"
taskQueue = "preprocessing"
workflowName = "preprocessing"

[worker]
maxConcurrentSessions = 1
```

Optional BagIt bag configuration:

```toml
[bagit]
checksumAlgorithm = "sha512"
```

### Enduro

The child workflow section for Enduro's configuration:

```toml
[[childWorkflows]]
type = "preprocessing"
namespace = "default"
taskQueue = "preprocessing"
workflowName = "preprocessing"
extract = false
sharedPath = "/home/preprocessing/shared"
```

## Local environment

The supported development workflow is to run `tilt up` from the Enduro
repository and load this repository through Enduro's `CHILD_WORKFLOW_PATHS`
mechanism.

Bring up the Enduro environment by following the [Enduro development manual].

### Set up

The specific requirements for this template are:

- clone this repository as a sibling of the Enduro repository
- configure `CHILD_WORKFLOW_PATHS=../preprocessing-base`
- configure `MOUNT_PREPROCESSING_VOLUME=true`
- run `tilt up` from the Enduro repository

All other development workflow details, including `.tilt.env`, live updates,
starting, stopping and clearing the environment, are documented in Enduro. This
repository can also provide local overrides through its own `.tilt.env` file,
including settings such as `TRIGGER_MODE_AUTO`.

### Requirements for development

While we run the services inside a Kubernetes cluster we recommend installing Go
and other tools locally to ease the development process.

- [Go] (1.24+)
- GNU [Make] and [GCC]

## Makefile

The Makefile provides developer utility scripts via command line `make` tasks.
Running `make` with no arguments (or `make help`) prints the help message.
Dependencies are downloaded automatically.

### Debug mode

The debug mode produces more output, including the commands executed. E.g.:

```shell
$ make env DBG_MAKEFILE=1
Makefile:10: ***** starting Makefile for goal(s) "env"
Makefile:11: ***** Fri 10 Nov 2023 11:16:16 AM CET
go env
GO111MODULE=''
GOARCH='amd64'
...
```

[Enduro development manual]: https://enduro.readthedocs.io/dev-manual/devel/
[go]: https://go.dev/doc/install
[make]: https://www.gnu.org/software/make/
[gcc]: https://gcc.gnu.org/
