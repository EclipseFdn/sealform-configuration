# sealform-configuration

Eclipse CSI [sealform](https://github.com/eclipse-csi/sealform) configuration to manage Eclipse Foundation project access to [SignPath.io](https://signpath.io).

sealform reads the YAML files in this repository and reconciles them with the SignPath organization through a `plan` / `apply` workflow. It manages projects, signing policies, artifact configurations and CI users. Certificates, users, groups and trusted build systems are only referenced by name, never modified.

## Layout

```
ef-projects/                      # configuration root (pass it with -c)
├── organization.yaml             # SignPath organization_id
├── projects/<slug>.yaml          # one file per project; the file name is the project slug
├── signing-policies/*.yaml       # shared signing policies, referenced with `extends:`
└── artifact-configurations/*.xml # shared artifact configurations, referenced with `file:`
```

## Setup

Install sealform (or prefix each command below with `uvx` instead):

```sh
uv tool install sealform        # or: pipx install sealform
```

sealform needs a SignPath API token. It reads the token from `--token-file FILE`, `--token-stdin` or the `SIGNPATH_API_TOKEN` environment variable, in that order. There is no option to pass the token on the command line.

```sh
export SIGNPATH_API_TOKEN=...
```

`validate` runs offline and needs no token. A Global Reader token is enough for `plan`. `apply` needs an administrator token.

## Usage

Run commands from the repository root and point `-c` at the configuration root:

```sh
sealform -c ef-projects validate   # offline: schema, references, XML
sealform -c ef-projects plan       # read-only: show what apply would change
sealform -c ef-projects apply      # apply the plan after confirmation
```

To limit `plan` or `apply` to some projects, repeat `-p`:

```sh
sealform -c ef-projects plan -p technology.apoapsis
```

## Adding a project

1. Copy an existing file in `ef-projects/projects/` to `<slug>.yaml`, where `<slug>` is the Eclipse project ID (for example `technology.foo`).
2. Set `name`, `repositories` and the CI user name (`<slug>-bot`), and choose an artifact configuration.
3. Run `validate`, then `plan`, and check the diff before you run `apply`.

SignPath returns a new CI user's API token only once. In an interactive session, `apply` prints it to the terminal. To save it to a file instead, run `apply --ci-token-file FILE` (the file is created with mode 0600). If the token is lost, run `sealform -c ef-projects ci-user rotate-token <name>` to generate a new one. The previous token stops working.

## Editor completion

To generate JSON Schemas for the configuration files:

```sh
sealform schema -o schemas
```

Run `sealform --help` and `sealform <command> --help` for the other options, including `import` and `--prune`.
