# Contributing

Installs and configures MySQL or MariaDB server on RHEL/CentOS, Debian/Ubuntu, or Arch Linux servers.

## Development setup

See [AGENTS.md](AGENTS.md#commands) for build/test/lint commands — not repeated here.

## Opening a PR

Use this repo's PR template — link the ticket (Jira/Linear/an issue) and describe how the change was verified.

## Before you open a PR

- [ ] Tests pass locally (`tests/test.sh` against the relevant distro/playbook combination)
- [ ] Lint/static analysis passes, if the repo has one configured
- [ ] Docs (`README.md`/`AGENTS.md`) updated if this changes how the role is built, run, or used
