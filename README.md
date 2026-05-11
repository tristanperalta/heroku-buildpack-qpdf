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

Set the qpdf version to install. Defaults to `12.3.2`.

```sh
heroku config:set QPDF_VERSION=12.3.2
```

Available versions can be found on the [qpdf releases page](https://github.com/qpdf/qpdf/releases). The buildpack uses the `qpdf-<version>-bin-linux-x86_64.zip` asset, so the version you pick must publish that artifact.

## How It Works

1. Downloads `qpdf-<version>-bin-linux-x86_64.zip` from [GitHub Releases](https://github.com/qpdf/qpdf/releases)
2. Caches the extracted tree between builds (keyed by version)
3. Installs binaries to `/app/.heroku/bin` and bundled shared libraries to `/app/.heroku/lib`
4. Writes `/app/.profile.d/qpdf.sh` to extend `PATH` and `LD_LIBRARY_PATH` at runtime

The release ships `qpdf` along with `fix-qdf`, `zlib-flate`, and a self-contained set of dependent libraries (`libqpdf`, `libjpeg`, `libgnutls`, `libnettle`, `libhogweed`, `libidn2`, `libtasn1`, `libunistring`, `libp11-kit`, `libffi`), so it does not rely on the stack image providing them.

## Architecture support

Only **x86_64** is supported. The qpdf project does not publish prebuilt aarch64 binaries on GitHub Releases, so the buildpack will fail fast on other architectures.

## Usage

After deploying, `qpdf` is available in your app's `PATH`:

```sh
heroku run qpdf --version
heroku run qpdf --linearize input.pdf output.pdf
```

The bundled libraries are picked up automatically via `LD_LIBRARY_PATH`.

## License

MIT
