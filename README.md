# github-workflows
A collection of reusable GitHub Actions and reusable Workflows for Java/Maven and Node (npm/pnpm). Designed to be easy to consume across repositories with clear, environment-driven versioning.

Contents
- Composite actions (reusable steps) under .github/actions
  - java-setup: sets up JDK using actions/setup-java@v4
  - maven-setup: prepares ~/.m2 and copies settings.xml
  - maven-build: runs Maven goals
  - node-setup: sets up Node via actions/setup-node@v4 and optional npm version
  - npm-install: installs dependencies using npm ci or npm install
  - node-build: runs npm build script
  - node-test: runs npm test (with extra args)
  - pnpm-setup: sets up Node and pnpm (pnpm/action-setup@v4)
  - pnpm-install: installs dependencies with pnpm
  - pnpm-build: runs pnpm build script
- Reusable workflows (workflow_call) under .github/workflows
  - maven.yml: Java + Maven build
  - node-npm.yml: Node (npm) build + test
  - node-pnpm.yml: Node (pnpm) build + test
- Maven settings template under .mvn/settings.xml

Versioning via environment variables
All versions can be controlled by environment variables or workflow_call inputs. Defaults are sensible and current.
- Java/Maven:
  - env JAVA_VERSION (default 21)
  - env JAVA_DISTRIBUTION (default temurin)
- Node/npm:
  - env NODE_VERSION (default 20)
  - env NPM_VERSION (optional)
- pnpm:
  - env PNPM_VERSION (default 9)

Using the reusable workflows
From any repository, you can call these workflows via workflow_call using the full path to this repo at a ref (tag/branch/commit). Example:

```yaml
name: CI (Maven)

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  maven:
    uses: spectrayan/github-workflows/.github/workflows/maven.yml@v1
    with:
      java-version: '21'
      goals: '-B -U clean verify'
      working-directory: '.'
    secrets:
      MAVEN_SERVER_USERNAME: ${{ secrets.MAVEN_SERVER_USERNAME }}
      MAVEN_SERVER_PASSWORD: ${{ secrets.MAVEN_SERVER_PASSWORD }}
```

```yaml
name: CI (Node with npm)

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  node:
    uses: spectrayan/github-workflows/.github/workflows/node-npm.yml@v1
    with:
      node-version: '20'
      npm-version: '10'
      install-ci: true
      build-script: 'build'
      test-script: 'test'
      test-args: '-- --coverage'
      working-directory: '.'
      registry-url: 'https://registry.npmjs.org'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

```yaml
name: CI (Node with pnpm)

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  node:
    uses: spectrayan/github-workflows/.github/workflows/node-pnpm.yml@v1
    with:
      node-version: '20'
      pnpm-version: '9'
      frozen-lockfile: true
      build-script: 'build'
      test-script: 'test'
      test-args: ''
      working-directory: '.'
      registry-url: 'https://registry.npmjs.org'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Using the composite actions directly
If you prefer to compose your own workflows, you can use the composite actions directly after checking out this repo with a relative path reference.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: spectrayan/github-workflows/.github/actions/java-setup@v1
        with:
          java-version: '21'
          distribution: 'temurin'
      - uses: spectrayan/github-workflows/.github/actions/maven-setup@v1
      - uses: spectrayan/github-workflows/.github/actions/maven-build@v1
        with:
          goals: '-B -U clean verify'
```

Security and credentials
- Maven: provide MAVEN_SERVER_USERNAME and MAVEN_SERVER_PASSWORD secrets to inject into .m2/settings.xml (placeholders __SERVER_USERNAME__ and __SERVER_PASSWORD__).
- NPM: provide NPM_TOKEN and optionally a registry-url to authenticate.

Repository layout
- .github/actions/*: composite actions per tool
- .github/workflows/*: reusable workflows
- .mvn/settings.xml: default Maven settings template

Notes
- All GitHub official actions are pinned to v4 where applicable (actions/checkout@v4, actions/setup-java@v4, actions/setup-node@v4).
- Node/npm caching is enabled via setup-node cache= npm, pnpm caching via cache= pnpm.
- You can override any version via inputs or env vars per your consuming workflow.
