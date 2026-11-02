# CI/CD Tools and Practices Final Project

**Project name:** `ci-cd-final-project`
**Author:** Karan Krishna K
**Course:** Continuous Integration and Continuous Delivery (CI/CD) — IBM Skills Network / Coursera

This repository contains the final project for the Coursera course **CI/CD Tools and Practices**. It takes the sample Hit Counter Flask service provided in the course and adds a complete CI/CD pipeline around it using GitHub Actions, Tekton tasks and OpenShift Pipelines.

## Application

The sample application is a small Flask REST API ("Hit Counter Service") that exposes:

| Method | Endpoint             | Description                      |
|--------|----------------------|----------------------------------|
| GET    | `/health`            | Health check                     |
| GET    | `/`                  | Service information              |
| GET    | `/counters`          | List all counters                |
| POST   | `/counters/<name>`   | Create a counter                 |
| GET    | `/counters/<name>`   | Read a counter                   |
| PUT    | `/counters/<name>`   | Increment a counter              |
| DELETE | `/counters/<name>`   | Delete a counter                 |

## Setup

After entering the lab environment you will need to run the `setup.sh` script in the `./bin` folder to install the prerequisite software.

```bash
bash bin/setup.sh
```

Then you must exit the shell and start a new one for the Python virtual environment to be activated.

```bash
exit
```

Run the linter and the unit tests locally with:

```bash
flake8 service --count --max-complexity=10 --max-line-length=127 --statistics
nosetests -v --with-spec --spec-color --with-coverage --cover-package=service
```

## Tasks

### 1. Continuous Integration with GitHub Actions

`.github/workflows/workflow.yml` defines a **CI workflow** that runs on every push and pull request to `main` inside a `python:3.9-slim` container:

1. **Checkout** the repository
2. **Install dependencies** from `requirements.txt`
3. **Lint with flake8**
4. **Run unit tests with nose** (with coverage reporting for the `service` package)

### 2. Tekton tasks

`.tekton/tasks.yml` defines the custom Tekton tasks used by the OpenShift pipeline:

- **cleanup** — empties the shared `source` workspace so every pipeline run starts from a clean slate
- **nose** — installs the Python dependencies and runs `nosetests` against the checked-out code

### 3. Continuous Delivery with OpenShift Pipelines

An OpenShift pipeline named **`oc-final`** wires the tasks together:

```
cleanup → git-clone → flake8 → nose → buildah → openshift-client (deploy)
```

- A **PersistentVolumeClaim** (`oc-lab-pvc`) backs the shared `pipeline-workspace` so the cloned source is available to every task
- **flake8** (from Tekton Hub) lints the code
- **nose** (custom task above) runs the unit tests
- **buildah** builds the container image from the `Dockerfile` and pushes it to the OpenShift internal registry
- **openshift-client** deploys the image with `oc create deployment` and exposes it with a route

## License

Licensed under the Apache License. See [LICENSE](/LICENSE)

## Author

Skills Network

## <h3 align="center"> © IBM Corporation 2023. All rights reserved. <h3/>
