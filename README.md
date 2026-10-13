# Heroku Buildpack for qpdf

A [Heroku buildpack](https://devcenter.heroku.com/articles/buildpacks) that installs the [qpdf](https://qpdf.sourceforge.io/) CLI and its bundled shared libraries on your Heroku app.

## Installation

```sh
heroku buildpacks:add https://github.com/tristanperalta/heroku-buildpack-qpdf
```

When using multiple buildpacks, add it before or after your primary buildpack as needed:

```sh
heroku buildpacks:add --index 1 https://github.com/tristanperalta/heroku-buildpack-qpdf
```

## Configuration

### `QPDF_VERSION`

Set the qpdf version to install. Defaults to `12.4.2`.

```sh
heroku config:set QPDF_VERSION=12.4.2
```

Available versions can be found on the [qpdf releases page](https://github.com/qpdf/qpdf/releases). The buildpack uses the `qpdf-<version>-bin-linux-<arch>.zip` asset and verifies it against the release's `qpdf-<version>.sha256` file, so the version you pick must publish both.

## How It Works

1. Downloads `qpdf-<version>-bin-linux-<arch>.zip` from [GitHub Releases](https://github.com/qpdf/qpdf/releases), where `<arch>` is `x86_64` or `aarch64` depending on the build machine
2. Verifies the download against the release's published sha256 checksum
3. Caches the extracted tree between builds (keyed by version and architecture)
4. Installs binaries to `/app/.heroku/bin` and bundled shared libraries to `/app/.heroku/lib`
5. Writes `/app/.profile.d/qpdf.sh` to extend `PATH` and `LD_LIBRARY_PATH` at runtime

The release ships `qpdf` along with `fix-qdf`, `zlib-flate`, and a self-contained set of dependent libraries (`libqpdf`, `libjpeg`, `libgnutls`, `libnettle`, `libhogweed`, `libidn2`, `libtasn1`, `libunistring`, `libp11-kit`, `libffi`), so it does not rely on the stack image providing them.

## Supported stacks

| Stack | Ubuntu | Status |
|-------|--------|--------|
| `heroku-26` | 26.04 | Tested |
| `heroku-24` | 24.04 | Tested |
| `heroku-22` | 22.04 | Tested |

The qpdf release bundle links against glibc 2.34 or newer, so any stack based on Ubuntu 22.04 or later works. The buildpack prints a warning on other stacks but still attempts the install.

## Architecture support

**x86_64** is supported and tested. **aarch64** (arm64) is supported for qpdf 12.4.0 and newer, which is when the qpdf project started publishing `bin-linux-aarch64.zip` assets; older versions fail with a clear error on arm64 build machines. Other architectures fail fast.

## Usage

After deploying, `qpdf` is available in your app's `PATH`:

```sh
heroku run qpdf --version
heroku run qpdf --linearize input.pdf output.pdf
```

The bundled libraries are picked up automatically via `LD_LIBRARY_PATH`.

## License

MIT
