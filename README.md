# duckdb-gcs

---

This DuckDB extension allows you to read files from Google Cloud Storage,
natively, using the Google Cloud C++ SDK.

DuckDB's core httpfs extension allows reading files from GCS, but it does so
using the S3 compatibility API. This is not as fast as the native API and it
requires using HMAC keys. HMAC keys are, by default, disabled by organization
policy for service accounts.

Note: Because the core `httpfs` extension registers itself as a handler for
`gs://` and `gcs://` URLs, this extension also supports `gcss://` as a way to
force its usage.

## Building

### Managing dependencies

DuckDB extensions uses VCPKG for dependency management. Enabling VCPKG is very
simple: follow the
[installation instructions](https://vcpkg.io/en/getting-started) or just run the
following:

```shell
git clone https://github.com/Microsoft/vcpkg.git
./vcpkg/bootstrap-vcpkg.sh
export VCPKG_TOOLCHAIN_PATH=`pwd`/vcpkg/scripts/buildsystems/vcpkg.cmake

```

### Build steps

Now to build the extension, run:

```sh
make
```

The main binaries that will be built are:

```sh
./build/release/duckdb
./build/release/test/unittest
./build/release/extension/gcs/gcs.duckdb_extension
```

- `duckdb` is the binary for the duckdb shell with the extension code automatically loaded.
- `unittest` is the test runner of duckdb. Again, the extension is already linked into the binary.
- `gcs.duckdb_extension` is the loadable binary as it would be distributed.

## Running the extension

To run the extension code, simply start the shell with `./build/release/duckdb`.

```
D CREATE SECRET gcpsecret (TYPE gcp, PROVIDER credential_chain);
D SELECT COUNT(*) FROM 'gcss://duckdb-gcs-testing/taxi_2019_04.parquet';
┌────────────────┐
│  count_star()  │
│     int64      │
├────────────────┤
│    7433139     │
│ (7.43 million) │
└────────────────┘
```

## Running the tests

Different tests can be created for DuckDB extensions. The primary way of testing
DuckDB extensions should be the SQL tests in `./test/sql`. These SQL tests can
be run using:

```sh
make test
```

### Installing the deployed binaries

To install extension binaries from S3, you will need to do two things.
Firstly, DuckDB should be launched with the `allow_unsigned_extensions` option
set to true. How to set this will depend on the client you're using. Some
examples:

CLI:

```shell
duckdb -unsigned
```

Python:

```python
con = duckdb.connect(':memory:', config={'allow_unsigned_extensions' : 'true'})
```

NodeJS:

```js
db = new duckdb.Database(':memory:', {"allow_unsigned_extensions": "true"});
```

Secondly, you will need to set the repository endpoint in DuckDB to the HTTP URL
of the bucket to install from. To do this run the following SQL query in DuckDB:

```sql
SET custom_extension_repository='https://duckdb-gcs-testing.s3.us-east-1.amazonaws.com/extensions';
```

The `duckdb-gcs-testing` bucket is a publicly-readable bucket that we host
builds of the extension in during development. Feel free to use your own bucket.

After running these steps, you can install and load the extension using the
regular INSTALL/LOAD commands in DuckDB:

```sql
INSTALL gcs
LOAD gcs
```

