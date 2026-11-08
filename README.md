# hashb

A lightweight content-addressable storage engine written in Rust, using BLAKE3 for content hashing.

## Usage

### Store a file

Store a file in the default content-addressable storage directory:

```bash
cargo run -- put <file_path>
```

If no file is specified, `hashb` reads from standard input:

```bash
echo "Hello, world!" | cargo run -- put
```

On success, `hashb` prints the object's BLAKE3 hash to standard output.

### Storage location

By default, `hashb` stores objects in the platform-specific application data directory:

| Platform | Default storage directory |
|---|---|
| Linux | `~/.local/share/hashb/` |
| macOS | `~/Library/Application Support/hashb/` |
| Windows | `%LOCALAPPDATA%\hashb\` |

The default storage directory is determined using the [`directories`](https://docs.rs/directories/latest/directories/struct.ProjectDirs.html) crate and is created automatically if it does not exist.

**Environment variable**

Override the default storage location using `HASHB_DEFAULT_STORAGE`:

```bash
export HASHB_DEFAULT_STORAGE="$HOME/my-hashb-store"
cargo run -- put <file_path>
```

If the configured directory does not exist, `hashb` creates it automatically.

**Custom storage directory**

Use the `--store` flag to specify a storage directory for an individual command:

```bash
cargo run -- --store <directory> put <file_path>
```

Unlike the default storage directory, a directory explicitly specified through `--store` must already exist.

Storage locations are resolved in the following order of precedence:

1. `--store` command-line argument
2. `HASHB_DEFAULT_STORAGE` environment variable
3. Platform-specific application data directory

## Storage structure

To avoid storing thousands of files in a single directory, `hashb` uses a two-character hash-prefix directory structure.

The first two characters of an object's 64-character BLAKE3 hash determine its parent directory, while the remaining 62 characters form the filename.

```text
hashb/
├── objects/
│   ├── a5/
│   │   └── 91a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e
│   └── e3/
│       └── b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
└── tmp/
```

Objects are written to temporary files before being persisted under their content hashes. Identical content produces the same object ID, allowing existing objects to be reused without duplication.

**Note:** Files are currently stored without compression. Optional compression support is planned for a future release.
