# Jsonwall

Jsonwall is the format of [Chisel](https://github.com/canonical/chisel)'s database.
This database records all the information about the packages, slices, paths, and
lists of paths under the slices installed in the file system by Chisel.

The Chisel DB shall be located at `/var/lib/chisel/manifest.wall` (according to
the definition of the 
[`base-files_chisel`](https://github.com/canonical/chisel-releases/blob/d938f025acc3fa4425999aa8ed558085296af0ff/slices/base-files.yaml#L74) slice),
as a `zstd` compressed file (the filename `manifest.wall` is enforced by Chisel
itself).

This repository is meant to be a "mirror" of the `jsonwall` module, which has a
different license than Chisel.

All bug reports and security issues should still be reported through the
[issues page](https://github.com/canonical/chisel/issues) and the
[security advisory](https://github.com/canonical/chisel/security/advisories) in
[Chisel's repository](https://github.com/canonical/chisel).

This repository currently tracks the release
[`v1.2.0`](https://github.com/canonical/chisel/releases/tag/v1.2.0)

## Format

### Header

The header contains three fields:

| Field       | Required | Type | Description                                                                       |
|-------------|----------|------|-----------------------------------------------------------------------------------|
| `jsonwall`  | True     | str  | The version of `jsonwall`.                                                        |
| `schema`    | True     | str  | The schema version that `Chisel` will use.                                        |
| `count`     | True     | int  | The number of JSON entries (or, lines) in this file, including the header itself. |

### Packages

For each package installed in the file system, a JSON object with `"kind":"package"`
must be present in the database. These JSON objects have the following attributes:

| Field       | Required | Type | Description                                              |
|-------------|----------|------|----------------------------------------------------------|
| `kind    `  | True     | str  | Type of JSON object. It must always be set to `package`. |
| `name`      | True     | str  | Name of the package.                                     |
| `version`   | True     | str  | Version of the package.                                  |
| `sha256`    | True     | str  | Digest of the package (in hex format).                   |
| `arch`      | True     | str  | Architecture of the package.                             |

### Slices
For each slice installed in the file system, a JSON object with `"kind":"slice"`
must be present in the database. These JSON objects have two attributes:

| Field       | Required   | Type | Description                                            |
|-------------|------------|------|--------------------------------------------------------|
| `kind    `  | True       | str  | Type of JSON object. It must always be set to `slice`. |
| `name`      | True       | str  | Name of the slice, in the `<pkg>_<slice>` format.      |

### Paths
For each path defined in the slice definitions that Chisel installs in the file
system, a JSON object with `"kind":"path"` must be present in the database.
These JSON objects have the following attributes:

| Field          | Required | Type | Description                                                                                                                                                                           |
|----------------|----------|------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `kind       `  | True     | str  | Type of JSON object. It must always be set to `path`.                                                                                                                                 |
| `path`         | True     | str  | Location of the path.                                                                                                                                                                 |
| `mode`         | True     | str  | The permissions of the path, in an octal value format.                                                                                                                                |
| `slices`       | True     | str  | The slices that have added this path.                                                                                                                                                 |
| `sha256`       | False    | str  | The original checksum of the file as in the package (in hex format). This attribute is required for all regular files, except the `manifest.wall` file itself, which is an exception. |
| `final_sha256` | False    | str  | The checksum of the file after it has been modified during installation (in hex format). This attribute is required only for files that have been mutated.                            |
| `size`         | False    | int  | The final size of the file, in bytes. This attribute is required for regular files, except the manifest.wall file itself, which is an exception.                                      |
| `link`         | False    | str  | The target, if the file is a symbolic link.                                                                                                                                           |

### List of Paths under a Slice

To state the path that a slice has added/changed, JSON objects with `"kind":"content"` are used. It has the following attributes:

| Field       | Required   | Type | Description                                              |
|-------------|------------|------|----------------------------------------------------------|
| `kind    `  | True       | str  | Type of JSON object. It must always be set to `content`. |
| `slice`     | True       | str  | Name of the slice.                                       |
| `path`      | True       | str  | Location of the path.                                    |
