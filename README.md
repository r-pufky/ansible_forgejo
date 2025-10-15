# Forgejo
Forgejo installation from public release tarball.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_forgejo/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_forgejo/tree/main/defaults/main)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_forgejo/blob/main/defaults/main/ports.yml)

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv) collection.

## Example Playbook
**STOP.**

WARNING
> Migrations from Gitea/Gogs in practice require manual intervention.
>
> ALWAYS backup databases and all repository data (or make a snapshot) before
> attempting a migration.
>
> Suggest creating a new container with data/db snapshots with Forgejo, then
> removing the old instance one confirmed working.
>
> [migration instructions](https://forgejo.org/docs/latest/admin/upgrade/from-gitea/).

Read defaults documentation.

Gitea migrations should be done **manually**: Gitea versions > `1.22` may not
be migratable. Only Forgejo `10.0.3` is guaranteed to safely migrate from Gitea
up to `1.22`.

This role explicitly manages each configuration parameter, and popular Gitea
roles will need to be re-evaulated before migration. ALWAYS BACKUP. See manual


Install 11.0.3 release of Forgejo; ensuring media files have proper
permissions. Version (and databases) will be migrated and updated on new
releases. Media permissions will be set to ensure Forgejo can read files.

``` yaml
- name: 'forgejo server'
  hosts: 'forgejo.example.com'
  become: true
  roles:
     - 'r_pufky.srv.forgejo'
  vars:
    forgejo_srv_version: 'v11.0.3'
    forgejo_srv_local_users:
      - username: 'admin_user'
        password: 'admin'
        email: 'admin@example.com'
        admin: true
    forgejo_cfg_custom_path: '/etc/forgejo'
    forgejo_cfg_app_work_path: '/srv/forgejo'
    forgejo_cfg_security_install_lock: true
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_docs/blob/main/ansible/environment.md)

Run all unit tests:
``` bash
molecule test --all
```

### Releases
Release format: **{OS}-{SERVICE}-{ROLE}**

Each type inherits the versioning system used; defaulting to schematic
versioning.

`12.0.0-2.0.3-1.0.0`

* 12.0.0 - Debian 12 (bookworm).
* 2.0.3 - Service/app version.
* 1.0.0 - Role version.

Releases are branched on Debian releases:

* **[13.x.x](https://github.com/r-pufky/ansible_forgejo)**: 13 Trixie.
* **[12.x.x](https://github.com/r-pufky/ansible_forgejo/tree/12.x)**: 12 Bookworm.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_forgejo/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
