<h1 align="center">Spark GitHub Actions</h1>

<p align="center"><em>Reusable GitHub Actions for building and deploying Spark-based projects.</em></p>

## Features

- **Runs in the Spark images** — every action executes inside the matching [Spark Docker image](https://hub.docker.com/u/jalendport), so CI uses the same PHP and Node builds as everywhere else.
- **Versions are explicit inputs** — every step names its own PHP or Node version, so nothing silently moves. A new runtime needs no change here, only a published image tag.
- **Composite, not Docker** — no per-run image build on the runner, so steps start immediately.
- **Correct file ownership** — containers run as the runner's own user, so `vendor/` and `node_modules/` never come back root-owned.
- **Dependency caching built in** — Composer and npm downloads are cached between runs without any extra steps in your workflow.
- **Deploy included** — SSH configuration, rsync, and a Laravel Forge trigger, covering the whole pipeline from checkout to deployed.

## Usage

### `composer-install`

Installs Composer dependencies, excluding dev dependencies, for deploy pipelines.

```yaml
- uses: jalendport/spark-github-actions/composer-install@master
  with:
    php: "8.3"
```

Runs `install --no-ansi --no-dev --no-interaction --no-progress --no-scripts --optimize-autoloader`.

### `composer-install-dev`

Installs Composer dependencies, including dev dependencies, for CI.

```yaml
- uses: jalendport/spark-github-actions/composer-install-dev@master
  with:
    php: "8.3"
```

### `composer-run`

Runs a Composer script.

```yaml
- uses: jalendport/spark-github-actions/composer-run@master
  with:
    script: phpstan
    php: "8.3"
```

### `npm-install`

Installs npm dependencies.

```yaml
- uses: jalendport/spark-github-actions/npm-install@master
  with:
    node: "20"
```

### `npm-run`

Runs an npm script.

```yaml
- uses: jalendport/spark-github-actions/npm-run@master
  with:
    script: build # optional, default "build"
    node: "20"
```

### `configure-ssh`

Writes an SSH key and host entry, then outputs the config alias for [`rsync-deploy`](#rsync-deploy) to deploy over.

```yaml
- name: Configure SSH
  id: configure-ssh
  uses: jalendport/spark-github-actions/configure-ssh@master
  with:
    environment: production # optional, default "production"
    ssh-host: ${{ secrets.PRODUCTION_SSH_HOST }}
    ssh-user: ${{ secrets.PRODUCTION_SSH_USER }}
    ssh-key: ${{ secrets.PRODUCTION_SSH_KEY }}
```

### `rsync-deploy`

rsyncs the workspace to a remote server over an alias from [`configure-ssh`](#configure-ssh).

```yaml
- uses: jalendport/spark-github-actions/rsync-deploy@master
  with:
    ssh-config: ${{ steps.configure-ssh.outputs.ssh-config }}
    ssh-path: ${{ secrets.PRODUCTION_SSH_PATH }}
    flags: "-avz --no-o --no-g --delete" # optional, this is the default
    excludes: |
      /.github
      /node_modules
```

### `trigger-laravel-forge`

POSTs to a Laravel Forge deployment trigger URL. A non-2xx response fails the step.

```yaml
- uses: jalendport/spark-github-actions/trigger-laravel-forge@master
  with:
    trigger-url: ${{ secrets.PRODUCTION_FORGE_DEPLOY_TRIGGER_URL }}
```

## Supported versions

The `php` and `node` inputs are required, and accept any tag published for the Spark images.

| Input  | Available                    |
| ------ | ---------------------------- |
| `php`  | 7.4, 8.0, 8.1, 8.2, 8.3, 8.4 |
| `node` | 16, 18, 20, 22, 24           |

## Deprecated paths

The version-per-directory actions are superseded by the version inputs above, and will be removed once no workflow references them.

| Deprecated                 | Replacement                       |
| -------------------------- | --------------------------------- |
| `composer-install/php-8.2` | `composer-install` + `php: "8.2"` |
| `composer-install/php-8.3` | `composer-install` + `php: "8.3"` |
| `npm-install/node-20`      | `npm-install` + `node: "20"`      |
| `npm-install/node-22`      | `npm-install` + `node: "22"`      |
| `npm-build/node-20`        | `npm-run` + `node: "20"`          |
| `npm-build/node-22`        | `npm-run` + `node: "22"`          |

The version input is required, so carry each pipeline's pinned version across when migrating.

## Support

Found a bug or have a question? [Open an issue](https://github.com/jalendport/spark-github-actions/issues).

---

<p align="center">Made by <a href="https://jalendport.com">Jalen Davenport</a></p>
