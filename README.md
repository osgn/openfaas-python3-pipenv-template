OpenFaaS Python Pipenv Template
=============================================

This template is for creating python3 based OpenFaas functions with a pipenv environment instead
of requirements.txt.  It runs the function as a non-root user and has some hooks
for prepping the python environment.

## Usage:

### Downloading the template

Using template pull with the repository's URL

```bash
faas-cli template pull https://github.com/visuas/openfaas-python3-pipenv-template
```

### Using the python3-pipenv template

Create a new function and prep pipenv environment

```bash
export FN="tester"
faas-cli new --lang python3-pipenv $FN
( cd $FN && pipenv sync )
```

### Build the function

```bash
faas-cli up -f $FN.yml
```

### Build, push, and deploy the function with extras

```bash
faas-cli up -f $FN.yml -o pillow
```

## Build args

Add to your function YML if applicable

### Setup a label pointing to the source repo for container registry:

```yaml
  SOURCE_REPO: https://github.com/apairmont/tester
```

### Perform some bootstrapping tasks in python:

```yaml
  PYTHON_BOOTSTRAP: import package; package.setup();
```