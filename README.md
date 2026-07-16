# seaweedfs

Installs and manages SeaweedFS

## Table of contents

- [Requirements](#requirements)
- [Default Variables](#default-variables)
  - [seaweedfs_actions](#seaweedfs_actions)
  - [seaweedfs_bin_dir](#seaweedfs_bin_dir)
  - [seaweedfs_component_state](#seaweedfs_component_state)
  - [seaweedfs_components](#seaweedfs_components)
  - [seaweedfs_config_dir](#seaweedfs_config_dir)
  - [seaweedfs_dir](#seaweedfs_dir)
  - [seaweedfs_download_base](#seaweedfs_download_base)
  - [seaweedfs_download_dir](#seaweedfs_download_dir)
  - [seaweedfs_filer_config](#seaweedfs_filer_config)
  - [seaweedfs_filer_sync_config](#seaweedfs_filer_sync_config)
  - [seaweedfs_master_config](#seaweedfs_master_config)
  - [seaweedfs_s3_config](#seaweedfs_s3_config)
  - [seaweedfs_user](#seaweedfs_user)
  - [seaweedfs_version](#seaweedfs_version)
  - [seaweedfs_volume_config](#seaweedfs_volume_config)
- [Dependencies](#dependencies)
- [License](#license)
- [Author](#author)

---

## Requirements

- Minimum Ansible version: `2.20`

## Default Variables

### seaweedfs_actions

List of actions the role does, accepts one or more actions.
Use comma without spaces as a delimiter for multiple actions.

**_Required:_** `true`<br />
**_Type:_** String<br />

#### Example usage

```YAML
  seaweedfs_actions: install
  seaweedfs_actions: install,disk_init,master_config,volume_config,filer_config,s3_config,filer_sync_config,component_state
```

### seaweedfs_bin_dir

PATH directory for the installed weed binary

**_Type:_** String<br />

#### Default value

```YAML
seaweedfs_bin_dir: /usr/local/bin
```

### seaweedfs_component_state

Target systemd state applied to every component listed in seaweedfs_components.

**_Required:_** `true`, only in case `seaweedfs_actions: component_state`<br />
**_Type:_** String<br />

#### Example usage

```YAML
  seaweedfs_component_state: started
  seaweedfs_component_state: restarted
```

### seaweedfs_components

List of seaweedfs components to apply seaweedfs_component_state to, accepts
one or more of: master, volume, filer, s3, filer_sync. Use comma without
spaces as a delimiter for multiple components. `volume` applies the state
to every seaweedfs-volume-* systemd service (one per disk in
seaweedfs_volume_config.volumes).

**_Required:_** `true`, only in case `seaweedfs_actions: component_state`<br />
**_Type:_** String<br />

#### Example usage

```YAML
  seaweedfs_components: master
  seaweedfs_components: master,volume,filer,s3,filer_sync
```

### seaweedfs_config_dir

Directory for seaweedfs component config/options files (e.g. master.conf, s3.json).

**_Type:_** String<br />

#### Default value

```YAML
seaweedfs_config_dir: /etc/seaweedfs
```

### seaweedfs_dir

Base directory for seaweedfs data, created with seaweedfs_user ownership
during the install action. Derives seaweedfs_master_dir
(seaweedfs_dir/master), seaweedfs_volume_dir (seaweedfs_dir/volumes) and
seaweedfs_filer_dir (seaweedfs_dir/filers) in vars/main.yml - each of these
subdirectories is created by its own component (master_config,
disk_init/volume_config, filer_config respectively), not by install.

**_Type:_** String<br />

#### Default value

```YAML
seaweedfs_dir: /var/lib/seaweedfs
```

### seaweedfs_download_base

Base URL to download installation artifacts

**_Type:_** String<br />

#### Default value

```YAML
seaweedfs_download_base: https://github.com/seaweedfs/seaweedfs/releases/download
```

### seaweedfs_download_dir

Destination directory for downloading the seaweedfs binary archive

**_Type:_** String<br />

#### Default value

```YAML
seaweedfs_download_dir: /tmp
```

### seaweedfs_filer_config

Configuration for the seaweedfs filer component. `weed_parameters`
(required) is a free-form dict of key/value pairs passed as-is into the
filer's `-options` file - keys match whatever flags `weed filer` accepts
(e.g. ip, master, port, metricsPort - see `weed filer -h`). `ip` and
`master` are required within it (host-specific - no default). Uses the
embedded leveldb2 filer store under `seaweedfs_filer_dir` (no filer.toml) -
matches the PoC's default setup. Rendered into an `-options` file under
`seaweedfs_config_dir` that the systemd unit passes to `weed`. Required
only when seaweedfs_actions includes `filer_config`.

**_Required:_** `true`, only when seaweedfs_actions includes `filer_config`<br />
**_Type:_** Dict<br />

#### Example usage

```YAML
seaweedfs_filer_config:
  weed_parameters:
    ip: 172.16.40.24
    master: 172.16.40.24:9333
    port: 8888
    metricsPort: 9330
```

### seaweedfs_filer_sync_config

Configuration for the seaweedfs filer.sync component - continuously
replicates a bucket/path between two filers (active-passive or
active-active). `weed_parameters` (required) is a free-form dict of
key/value pairs passed as-is into the filer-sync's `-options` file - keys
match whatever flags `weed filer.sync` accepts (e.g. a, a.path, b, b.path,
isActivePassive, metricsPort - see `weed filer.sync -h`). `a` and `b` (the
two filer addresses being synced, e.g. "172.16.40.24:8888") are required
within it. Rendered into an `-options` file under seaweedfs_config_dir that
the systemd unit passes to `weed`. The systemd unit requires the local
seaweedfs-filer.service to be configured and running. Required only when
seaweedfs_actions includes `filer_sync_config`.

**_Required:_** `true`, only when seaweedfs_actions includes `filer_sync_config`<br />
**_Type:_** Dict<br />

#### Example usage

```YAML
seaweedfs_filer_sync_config:
  weed_parameters:
    a: 172.16.40.24:8888
    a.path: /buckets/my-test-bucket
    b: 172.16.80.24:8888
    b.path: /buckets/my-test-bucket
    isActivePassive: true
    metricsPort: 9332
```

### seaweedfs_master_config

Configuration for the seaweedfs master component. `weed_parameters`
(required) is a free-form dict of key/value pairs passed as-is into the
master's `-options` file - keys match whatever flags `weed master` accepts
(e.g. ip, port, volumeSizeLimitMB, metricsPort - see `weed master -h`).
`ip` is required within it (host-specific - no default; the address the
master binds to and advertises to volumes/filers). The master's data
directory (`mdir`) is always injected automatically from
`seaweedfs_master_dir` and must not be set in weed_parameters. Rendered
into an `-options` file under `seaweedfs_config_dir` that the systemd unit
passes to `weed`, rather than being passed as individual CLI flags.
Required only when seaweedfs_actions includes `master_config`.

**_Required:_** `true`, only when seaweedfs_actions includes `master_config`<br />
**_Type:_** Dict<br />

#### Example usage

```YAML
seaweedfs_master_config:
  weed_parameters:
    ip: 172.16.40.24
    port: 9333
    volumeSizeLimitMB: 1000
    metricsPort: 9327
```

### seaweedfs_s3_config

Configuration for the seaweedfs S3 gateway component. `weed_parameters`
(required) is a free-form dict of key/value pairs passed as-is into the S3
gateway's `-options` file - keys match whatever flags `weed s3` accepts
(e.g. filer, port, ip.bind, metricsPort - see `weed s3 -h`). `filer` is
required within it (comma-separated filer address(es) the gateway proxies
to). The `config` key (path to s3.json) is always injected automatically
and must not be set in weed_parameters. `identities` (required) is the raw
list of SeaweedFS identity objects (name, credentials: [{accessKey,
secretKey}], actions) written verbatim into `s3.json` - same schema as
`weed scaffold -config=s3` / the PoC's s3.json. Credentials are currently
plain text (test environment only; revisit with Vault before using this in
production). Required only when seaweedfs_actions includes `s3_config`.

**_Required:_** `true`, only when seaweedfs_actions includes `s3_config`<br />
**_Type:_** Dict<br />

#### Example usage

```YAML
seaweedfs_s3_config:
  weed_parameters:
    filer: 172.16.40.24:8888
    port: 8333
    ip.bind: 0.0.0.0
    metricsPort: 9331
  identities:
    - name: poc-admin
      credentials:
        - accessKey: changeme-access-key
          secretKey: changeme-secret-key
      actions:
        - Admin
        - Read
        - List
        - Tagging
        - Write
```

### seaweedfs_user

Linux user name to own seaweedfs directories and run the weed process.
Must already exist on the target host (e.g. provisioned via the linux_user role).

**_Type:_** String<br />

#### Default value

```YAML
seaweedfs_user: seaweedfs
```

### seaweedfs_version

SeaweedFS release version to install (without leading "v")

**_Type:_** String<br />

#### Default value

```YAML
seaweedfs_version: 4.38
```

### seaweedfs_volume_config

Configuration for the seaweedfs volume component. `volumes` is a list of
disks, each with: `block_device` (e.g. /dev/sdb), `label` (disk label type,
e.g. gpt), `file_system` (e.g. ext4), `mount_dir` (a bare directory name,
not a full path - rendered as `seaweedfs_volume_dir/<mount_dir>`, e.g.
mount_dir "disk1" -> /var/lib/seaweedfs/volumes/disk1), `mount_options`
(comma-separated fstab options, e.g. defaults,noatime) - all required only
when seaweedfs_actions includes `disk_init` - plus a per-volume
`weed_parameters` dict (free-form, e.g. port, metricsPort, max - unique per
disk, required only when seaweedfs_actions includes `volume_config`). Each
block_device is consumed entirely as a single partition (100%) - seaweedfs
volumes do not share disks.
The top-level `weed_parameters` dict (required, e.g. ip, master) is shared
across every volume on the host. For each volume it is combined with that
volume's own `weed_parameters` (per-volume keys win on conflict) plus an
automatically injected `dir` (derived from
`seaweedfs_volume_dir/<mount_dir>` and must not be set in weed_parameters)
before being rendered into that volume's `-options` file - keys match
whatever flags `weed volume` accepts (see `weed volume -h`). Each volume
gets its own systemd unit
(`seaweedfs-volume-<mount_dir>`), e.g. mount_dir "disk1" ->
seaweedfs-volume-disk1. Required only when seaweedfs_actions includes
`volume_config` (and `volumes` alone when seaweedfs_actions includes
`disk_init`, or `component_state` with `volume` listed in
seaweedfs_components).

**_Required:_** `true`, only when seaweedfs_actions includes `disk_init`, `volume_config`, or `component_state` with `volume` in seaweedfs_components<br />
**_Type:_** Dict<br />

#### Example usage

```YAML
seaweedfs_volume_config:
  weed_parameters:
    ip: 172.16.40.24
    master: 172.16.40.24:9333
  volumes:
    - block_device: /dev/sdb
      label: gpt
      file_system: ext4
      mount_dir: disk1
      mount_options: defaults,noatime
      weed_parameters:
        port: 8080
        metricsPort: 9328
```

## Dependencies

None.

## License

MIT

## Author

freedform
