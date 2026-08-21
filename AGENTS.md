# ansible-role-mysql

Installs and configures MySQL or MariaDB server on RHEL/CentOS, Debian/Ubuntu, or Arch Linux hosts.

## Architecture in a paragraph

Playbooks that need a database server pull in this role to get one installed and configured the
same way regardless of target distro. It's a standalone Ansible role with no role dependencies
(`meta/main.yml`): `tasks/main.yml` first branches on `ansible_os_family` to run one of
`setup-RedHat.yml`, `setup-Debian.yml`, or `setup-Archlinux.yml`, each of which installs the
right packages using the OS-specific names and paths defined in `vars/<OSFamily>.yml` (package
list, daemon name, config file/socket/PID paths, log locations). Once installed, the OS-agnostic
tasks take over: `configure.yml` renders `templates/my.cnf.j2` from `defaults/main.yml`
variables, `secure-installation.yml` sets the root/user passwords and strips anonymous users and
the test database, `databases.yml` and `users.yml` provision `mysql_databases`/`mysql_users`, and
`replication.yml` wires up master/slave replication when `mysql_replication_role` is set. Every
behavior a consuming playbook can control is a `defaults/main.yml` variable — there's no other
configuration surface.

## File map

```
defaults/
  main.yml               # every consumer-facing variable, with inline comments
handlers/
  main.yml                # restarts {{ mysql_daemon }} (name varies by OS/vars file)
meta/
  main.yml                # Galaxy metadata; dependencies: []
tasks/
  main.yml                 # entry point: OS dispatch, then the shared configure/secure/db/user/replication chain
  variables.yml             # include_vars for the matching vars/<OSFamily>.yml
  setup-RedHat.yml          # RHEL/CentOS package install
  setup-Debian.yml          # Debian/Ubuntu package install (stops mysql + clears innodb logs after first install)
  setup-Archlinux.yml       # Arch package install
  configure.yml             # renders templates/my.cnf.j2
  secure-installation.yml   # root/user password, anonymous users, test db removal
  databases.yml             # mysql_databases -> mysql_db module
  users.yml                 # mysql_users -> mysql_user module
  replication.yml           # master/slave replication setup
templates/
  my.cnf.j2                 # global my.cnf
  root-my.cnf.j2             # ~/.my.cnf for mysql_root_username
  user-my.cnf.j2              # ~/.my.cnf for mysql_user_name (when different from root)
vars/
  Archlinux.yml, Debian.yml, RedHat-6.yml, RedHat-7.yml   # per-OS package names, paths, daemon name
tests/
  test.yml                  # generic test playbook (role_under_test)
  centos-7-test.yml         # CentOS 7 variant (overrides to MariaDB package names/paths)
  initctl_faker              # fake initctl binary, needed on Ubuntu 14.04 containers
  README.md                   # manual test-run instructions
.travis.yml                # CI matrix; test shim is fetched at run time from a gist on geerlingguy's account (see Conventions)
```

## Commands

Test (requires Docker; the runner script is gitignored and fetched at run time, matching
`.travis.yml`):

```bash
wget -O tests/test.sh https://gist.githubusercontent.com/geerlingguy/73ef1e5ee45d8694570f334be385e181/raw/
chmod +x tests/test.sh
distro=centos7 playbook=centos-7-test.yml ./tests/test.sh
# or: distro=[centos6|debian8|ubuntu1604|ubuntu1404] playbook=test.yml ./tests/test.sh
```

Set `cleanup=false container_id=$(date +%s)` first if you want the container left running for
inspection after the playbook runs.

No lint config (`.ansible-lint`, `.yamllint`) exists in this repo.

## Conventions

- `README.md` states the license as `MIT / BSD`; `meta/main.yml` says `"license (BSD, MIT)"` (an
  unedited Galaxy scaffold placeholder), while the actual `LICENSE` file is MIT-only. Confirm the
  real license before relying on any of the three.
- All per-OS values (package names, daemon name, config/socket/PID paths, log locations) live in
  `vars/<OSFamily>.yml`, never hardcoded in tasks — add a new OS by adding a `vars/` file and a
  `setup-<OSFamily>.yml`, not by branching inside the shared tasks.
- `mysql_root_password_update` / `mysql_user_password_update` gate password changes so the role
  stays idempotent — a password is only reset on first install or when the flag is explicitly set.
  Don't remove that guard when touching `secure-installation.yml`.
- `.travis.yml`'s test script downloads its test shim (`tests/test.sh`, gitignored) from a gist
  hosted on geerlingguy's personal GitHub account rather than a file committed to this repo —
  Travis runs here depend on that gist staying available.

## See also

- [README.md](README.md) — install and usage
- [CONTRIBUTING.md](CONTRIBUTING.md) — PR process
