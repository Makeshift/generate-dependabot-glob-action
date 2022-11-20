# Generate Dependabot Glob Action

This action creates a `dependabot.yml` file from a user-provided template by replacing instances of directory globs with an array of objects matching that glob, with all the other keys copied.

For example, the following template:

```yaml
  - package-ecosystem: 'docker'
    directory: '/test/docker/*/Dockerfile*'
    schedule:
      interval: 'daily'
```

Will result in:

```yaml
  - package-ecosystem: 'docker'
    directory: '/test/docker/container_1/'
    schedule:
      interval: 'daily'
  - package-ecosystem: 'docker'
    directory: '/test/docker/container_2/'
    schedule:
      interval: 'daily'
  - package-ecosystem: 'docker'
    directory: '/test/docker/weird_dockerfile/'
    schedule:
      interval: 'daily'
```

Note that the basename of any matching directory is used as the value.

This action uses the [glob](https://www.npmjs.com/package/glob) node module. Refer to its documentation for more information on the glob syntax.

The default configuration for `glob` is as follows:

```js
const globOpts = {
  root: process.cwd(),
  mark: true,
  matchBase: true,
  nomount: true,
  follow: core.getInput('follow-symbolic-links') === 'true' || true
}
```

If these options are not sufficient, please open an issue and let me know.

## Quickstart

### Create a `.github/dependabot.template.yml` file
  
  ```yaml
version: 2

updates:
  - package-ecosystem: 'github-actions'
    directory: '/'
    schedule:
      interval: 'daily'

  - package-ecosystem: 'docker'
    directory: '/test/docker/*/Dockerfile*'
    schedule:
      interval: 'weekly'

  - package-ecosystem: 'npm'
    directory: '/test/npm/*/{package-lock.json,yarn.lock}'
    schedule:
      interval: 'daily'

  ```

### Create a `.github/workflows/generate_dependabot.yml` file

Note that this action does not create a PR or otherwise commit the generated file. You will need to do that yourself.

```yaml
name: Generate dependabot.yml

on:
  push:
  repository_dispatch:
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      
      - uses: actions/checkout@v3
        
      - name: Generate dependabot.yml
        uses: Makeshift/generate-dependabot-glob-action@v1

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v4
```

Done.
