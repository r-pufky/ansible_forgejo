# [Forgejo][h]
Forgejo installation from public release tarball.

## [Requirements][i]
Requires [r_pufky.srv][g] galaxy-ng collection. See
[additional documentation][m] and [reference documentation][h] for
troubleshooting and config variables.

Install size: ~162MB

> Only Forgejo **10.0.3** is guaranteed to migrate from Gitea up to **1.22**.

> Tasks [potentially touching Network Mounted Filesystems][q] will be run as
> the task user and fallback to the service user. Manage these locations
> externally if these fail.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage
[Postgres and MySql databases must be setup][l] before role is applied and are
highly recommended over SQLite for production use. Database backend migrations
(SQLite/Postgres/MySQL) must be done manually.

### WARNING
> Place user data **locations** outside of role location and manage externally.
>
> A custom WORK_PATH is automatically detected with forgejo_cfg_file and
> service is updated accordingly. WORK_PATH outside of /var/opt/forgejo must be
> managed manually.
>
> Manage forgejo_srv_user, forgejo_srv_group externally if additional SSH
> configuration needs to be done. See [role user defaults][n].

  Path                    | Usage
 -------------------------|-------
 /etc/opt/forgejo/app.ini | forgejo_cfg_file always deployed here.
 /var/opt/forgejo         | Forgejo working directory, role user home directory.
 /var/opt/forgejo/cfg     | forgejo_cfg_d always deployed here.
 /srv/forgejo             | Auto-created location if forgejo_cfg_file is not set.
 {AppWorkPath}/backup     | forgejo_flg_backup backups are stored here.

### NOTE
> Forgejo uses jinja2 templating to parse app.ini and other files. Templates
> are automatically used when **forgejo_cfg_file** or **forgejo_cfg_d** are
> used.
>
> Use **raw** feature to allow jinja templating to pass through to rendered
> template:

``` ini
[badges]
GENERATOR_URL_TEMPLATE = {% raw %}'{{.label}}-{{.text}}'{% endraw %}

```

Renders app.ini as:
``` ini
[badges]
GENERATOR_URL_TEMPLATE = '{{.label}}-{{.text}}'
```

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                    | Notes
 ------|-------------------------|-------
  1    | forgejo_flg_backup      | Create backup before making changes?
  2    | forgejo_flg_maintenance | Preform role maintenance tasks.
  3    | forgejo_flg_install     | Install required packages, users, etc.
  4    | forgejo_flg_config      | Install user-defined config.

### Example Playbooks

#### New deployment
Forgejo will be ready to use, allowing logins with **admin_test**. This is
good to get an initial working deployment to further configure.

``` yaml
- name: 'Create new Forgejo instance using SQLite3, creating admin user.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.forgejo'
  vars:
    forgejo_cfg_users:
      - username: 'admin_test'
        password: 'adminadmin'
        email: 'admin@example.com'
        admin: true
```

> **/etc/opt/forgejo/app.ini** should be copied to the ansible controller and
> secret keys added to ansible vault for
> [Pre-configured deployments](#pre-configured-deployment).

#### Pre-configured deployment
Forgejo deployments will typically use this. It will deploy a pre-configured
app.ini and optionally ansible-sourced templated files to be used by the
Forgejo service.

Forgejo configuration is complex and nuanced. See [Config Cheet Sheet][o] and
[Example app.ini][p] keeping in mind where the [role places files](#usage).

Example app.ini.j2
``` ini
; Example app.ini.j2 to source. All non-role directories (/d) are managed
; externally. Configured to use an existing postgres database backend, load
; secrets from ansible vault and pass through a jinja2 template to forgejo for
; custom badge URL.
;
; See templates/default/app.ini for more examples.
WORK_PATH = /var/opt/forgejo

[repository]
ROOT = /d/hosted/repos

[ui]
SHOW_USER_EMAIL = false

[server]
ROOT_URL = http://192.168.1.1:3000/
DISABLE_SSH = true
APP_DATA_PATH = /d/hosted/storage/app-data
LFS_START_SERVER = true
LFS_JWT_SECRET = {{ vault_forgejo_lfs_jwt_secret }}

[database]
DB_TYPE = postgres
HOST = 192.168.1.1:5432
NAME = forgejo
USER = forgejo
; Backticks are required in case password contains special characters.
PASSWD = `{{ vault_forgejo_postgres_pass }}`
SSL_MODE = disable

[security]
; Marks installation as complete, disabling the initial configuration screen.
INSTALL_LOCK = true
SECRET_KEY = {{ vault_forgejo_secret_key }}
INTERNAL_TOKEN = {{ vault_forgejo_internal_token }}

[openid]
ENABLE_OPENID_SIGNIN = false
ENABLE_OPENID_SIGNUP = false

[service]
DISABLE_REGISTRATION = true
REQUIRE_SIGNIN_VIEW = true
DEFAULT_ALLOW_CREATE_ORGANIZATION = false
NO_REPLY_ADDRESS = noreply.example.org
SHOW_REGISTRATION_BUTTON = false

[mailer]
; Not complete configuration of mailer.
; Only example of forgejo_cfg_d for certs.
CLIENT_CERT_FILE = /var/opt/forgejo/cfg/cert.pem
CLIENT_KEY_FILE = /var/opt/forgejo/cfg/key.pem

[log]
LEVEL = Warn

[git.timeout]
MIGRATE = 1200
MIRROR = 1200

[oauth2]
JWT_SECRET = {{ vault_forgejo_jwt_secret }}

[storage]
PATH = /d/hosted/storage

; Use raw for jinja templates that should be passed to forgejo.
[badges]
GENERATOR_URL_TEMPLATE = {% raw %}'{{.label}}-{{.text}}'{% endraw %}
```

``` yaml
- name: 'Deploy Forgejo using Postgres, setting admin user, using deployed certs.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.forgejo'
  vars:
    forgejo_flg_config: true
    forgejo_cfg_file: 'host_vars/forgejo.example.com/data/app.ini.j2'
    forgejo_cfg_d: 'host_vars/forgejo.example.com/data/work'
    forgejo_cfg_users:
      - username: '{{ vault_forgejo_admin_user }}'
        password: '{{ vault_forgejo_admin_pass }}'
        email: '{{ vault_forgejo_admin_email }}'
        admin: true
```

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

Testing variables:

  Variable            | type | Description
 ---------------------|------|-------------
  molecule_flg_inject | bool | Flag to inject files locally.

### [Releases][b]

  Release | Debian | Ansible | Forgejo  | Notes
 ---------|--------|---------|----------|-------
  4.x.x   | 13     | 2.20    | v11.0.11 | Ansible 2.20, feature flags, and semantic versioning.
  3.x.x   | 13     | 2.18    | v11.0.3  | Migrate to Debian Trixie.
  2.x.x   | 12     | 2.18    | v11.0.3  | Update to 11 LTS with fully manage configuration.
  1.x.x   | 12     | 2.18    | v10.0.3  | Implement data annotations.
  0.x.x   | 12     | 2.18    | v10.0.3  | Initial release.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]

[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_forgejo/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_srv
[h]: https://forgejo.org
[i]: https://github.com/r-pufky/ansible_forgejo/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_forgejo/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_forgejo/blob/main/defaults/main/ports.yml
[l]: https://forgejo.org/docs/next/admin/installation/database-preparation
[m]: https://r-pufky.github.io/docs/vcs/forgejo
[n]: https://github.com/r-pufky/ansible_forgejo/tree/main/vars/main.yml
[o]: https://forgejo.org/docs/latest/admin/config-cheat-sheet/
[p]: https://codeberg.org/forgejo/forgejo/src/branch/forgejo/custom/conf/app.example.ini
[q]: https://r-pufky.github.io/ansible_docs/best_practice/patterns/#network-mounts