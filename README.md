# lms-githubaction-library
Library of github action workflows

## Release Instructions

```
# substitute in the correct version for this specific tag
git tag -a -m"tagging action release" v1.0.X

# always tag with just the major version to update to the latest
git tag -a -m"tagging action release" v1 --force

git push --tags --force
```

## Usage

### Release Job

See [release.yml](.github/workflows/release.yml) for the various `inputs` this action supports (or [below](#inputs)).

```yml
# Description
# -----------
# This workflow builds and releases the maven artifact
#
# Setup
# -----
# 1. Create the following secrets inside GitHub:
#    - LMS_GIT_CONFIG (Base64 encoded .gitconfig file)

name: Maven Release

on:
  workflow_dispatch:
jobs:
  release:
    uses: iu-uits-lt/lms-githubaction-library/.github/workflows/release.yml@v1
    with:
      java_version: 25
    secrets:
      LMS_GIT_CONFIG: ${{ secrets.LMS_GIT_CONFIG }}

```

#### Inputs

| Input | Required? | Default | Description |
| ----- | --------- | ------- | ----------- |
| `java_version` | `true` | `"25"` | Java version to use |

#### Secrets

| Secret | Description |
| ----- | --------- |
| `LMS_GIT_CONFIG` | BBase64 encoded .gitconfig file |

### Deploy Job

See [kube-deploy.yml](.github/workflows/kube-deploy.yml) for the various `inputs` this action supports (or [below](#inputs-1)).

It's assumed that the project in question has in its pom, a profile for `build-image` which will generate and push a Dockerfile to the image registry.

```yml
# Description
# -----------
# This workflow builds and releases the Docker image to Harbor, then deploys out to the kubernetes environment.
#
# Setup
# -----
# 1. Create the following secrets inside GitHub:
#    - LMS_GH_TOKEN (Personal access token for the lmsgit user that has read access to other repositories)
#    - LMS_MAVEN_SETTINGS (Base64 encoded settings.xml file)
#    - LMS_REGISTRY_PASSWORD (Registry password - push access)
#    - LMS_REGISTRY_USERNAME (Registry username)
# 2. Create the following environments, representative of the deployments:
#    - reg
#    - stg
#    - prd
# 3. Create the following secrets in each of the above environments
#    - LMS_KUBECONFIG_DEPLOYER (Base64 encoded kubernetes deployment service account for the appropriate cluster)
# 4. Create the following variables in each of the above environment
#    - DOCKER_TAG (tag name for the docker image that will be deployed i.e. stable, unstable-reg, etc)
# 5. Create the following variables in the repository
#    - IMAGE_REPO_NAME (Harbor registry name where the docker image will be pushed)
#    - DEPLOY_DIR (Directory where the helm deployment files can be found)
#    - JAR_FILE (Name of the application jar file)
#    - K8S_RELEASE_PREFIX (Kubernetes release name prefix for the deployed application)
#    - BUILD_PROFILES (Comma separated list of maven build profiles that need to be activated)

name: Build and Deploy

on:
  workflow_dispatch:
    inputs:
      server_env:
        type: environment
        required: true
        description: Select deployment env
        default: reg
      helm_deployer_branch:
        required: true
        description: Indicate the branch that contains the helm deployment files
        default: develop

jobs:
  mvn_build_deploy:
    uses: iu-uits-lt/lms-githubaction-library/.github/workflows/kube-deploy.yml@v1
    with:
      java_version: 25
      server_env: ${{ github.event.inputs.server_env }}
      helm_deployer_branch: ${{ github.event.inputs.helm_deployer_branch }}
    secrets:
      LMS_MAVEN_SETTINGS: ${{ secrets.LMS_MAVEN_SETTINGS }}
      LMS_GH_TOKEN: ${{ secrets.LMS_GH_TOKEN }}
      LMS_REGISTRY_USERNAME: ${{ secrets.LMS_REGISTRY_USERNAME }}
      LMS_REGISTRY_PASSWORD: ${{ secrets.LMS_REGISTRY_PASSWORD }}
      LMS_KUBECONFIG_DEPLOYER: ${{ secrets.LMS_KUBECONFIG_DEPLOYER }}

```

#### Inputs

| Input | Required? | Default | Description |
| ----- | --------- | ------- | ----------- |
| `java_version` | `true` | `"25"` | Java version to use |
| `server_env` | `true` | `"reg"` | Environment to deploy to |
| `helm_deployer_branch` | `true` | `"develop"` | Branch that contains the helm deployment files |

#### Secrets

| Secret | Description |
| ----- | ----------- |
| `LMS_MAVEN_SETTINGS` | Base64 encoded settings.xml file for Maven |
| `LMS_GH_TOKEN` | GitHub token for repository access |
| `LMS_REGISTRY_USERNAME` | Username for Docker registry |
| `LMS_REGISTRY_PASSWORD` | Password for Docker registry |
| `LMS_KUBECONFIG_DEPLOYER` | Base64 encoded kubeconfig file for deployment |

### Maven Tests Job

See [maven.yml](.github/workflows/maven.yml) for the various `inputs` this action supports (or [below](#inputs-2)).

```yml
# This workflow will build a Java project with Maven, and cache/restore any dependencies to improve the workflow execution time
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-maven

name: Java CI with Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    uses: iu-uits-lt/lms-githubaction-library/.github/workflows/maven.yml@v1
    with:
      java_version: 25

```

#### Inputs

| Input | Required? | Default | Description |
| ----- | --------- | ------- | ----------- |
| `java_version` | `true` | `"25"` | Java version to use |

#### Secrets

None

### CodeQL Analysis Job

See [codeql-analysis.yml](.github/workflows/codeql-analysis.yml) for the various `inputs` this action supports (or [below](#inputs-3)).

```yml
# For most projects, this workflow file will not need changing; you simply need
# to commit it to your repository.
#
# You may wish to alter this file to override the set of languages analyzed,
# or to provide custom queries or build logic.
#
# ******** NOTE ********
# We have attempted to detect the languages in your repository. Please check
# the `language` matrix defined below to confirm you have the correct set of
# supported CodeQL languages.
#
name: "CodeQL"

on:
  push:
    branches: [ main ]
  pull_request:
    # The branches below must be a subset of the branches above
    branches: [ main ]

jobs:
  analyze:
    uses: iu-uits-lt/lms-githubaction-library/.github/workflows/codeql-analysis.yml@v1
    with:
      java_version: 25
      languages: "[ 'java', 'javascript' ]"

```

#### Inputs

| Input | Required? | Default | Description |
| ----- | --------- | ------- | ----------- |
| `java_version` | `true` | `"25"` | Java version to use |
| `languages` | `false` | `["java","javascript"]` | Languages to perform analysis on |

#### Secrets

None
