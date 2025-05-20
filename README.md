# Camelot SDK for nucleo-u5a5zj development board

This repository is the SDK generation repository for the STM32 nucleo-u5a5zj board,
allowing to rebuild a complete SDK release of the Camelot OS for this board.

The content of the SDK is stored in the project.toml file, listing overall components part of the SDK.
SDK components configuration for the current board is set in the `configs` directory, while the current board
device tree is set in the `dts` directory.

## SDK build-dependencies (Debian/ubuntu)

Install the (Debian/Ubuntu) dependencies :
```
apt install meson doxygen srecord pandoc python3-pip pkg-config gcc-arm-none-eabi curl device-tree-compiler git
```

Clone the SDK : 
```
git clone https://github.com/camelot-os/camelot-sdk.git
```

Create a new Python3 virtual env for the Camelot SDK:
```
cd camelot-sdk
python3 -m venv .venv
source .venv/bin/activate
```

Download the python dependencies first, including the barbican Camelot project management tool:
```
pip install -r requirements.txt
```

Be sure to have the C cross-toolchain and the Rust toolchain for the Cortex-M33 target. To do so, first create a cross directory for meson :
```
mkdir -p /home/$USER/.local/share/meson/cross
```

Copy the chosen .ini file into the newly created directory:
```
cp armv8m-mainline/arm-none-eabi-gcc.ini /home/$USER/.local/share/meson/cross/
```

Edit the copied file and update the cross_toolchain value to '/usr/' like this:
```
cross_toolchain = '/usr/'
```

Install the rustup stable toolchain :
```
curl https://sh.rustup.rs -sSf | sh -s -- --default-toolchain stable  -y
```

Once installed, source the env file :
```
source "$HOME/.cargo/env"
```

Install the rust thumbv8m.main-none-eabi target :
```
rustup target add thumbv8m.main-none-eabi
```

Barbican also requires cargo-index crate in order to manipulate Rust package properly to be included as a
fully standalone, offline, SDK, please install it using cargo:

```
cargo install cargo-index
```

## Building the SDK

The SDK can now be built using the barbican camelot project tool.

First download the SDK components, including, at least but not limited to, the Camelot kernel and runtime.

```
barbican download
```

Once sources are downloaded, setup the SDK build configuration:
```
barbican setup
```

To finish with, build the SDK:
```
ninja -C output/build
```


## Using the SDK

### Using pre-built SDK package

The pre-built package for this SDK is automatically deployed as a Conan package in the Jfrog conan center with the name
`camelot-sdk-nucleo-u5a5zj` and can be downloaded and consume as-is using the Conan tooling.

Once the SDK has been built, just run, for example:

```
conan export-pkg . -s os="Linux" -s arch="x86_64"
```

The conan package is then generated and loaded to the current cache. If you used a Conan repository (like from a
Jfrog artifactory instance for e.g.) you can publish this package in the artifact manager.
Such a job sould be used by the CD automatically in a production environment.

### Using locally built SDK package

The effective SDK camelot, that can be stored in a tarball or any container, is generated in the `output/staging` directory.
This directory can be packaged using any container, including tarball, to be delivered, or through any artifact management tools.
The SDK is standalone, except for toolchains that are not yet a part of it.

When building a C application, the pkg_config directory is in the `usr/local/lib/pkgconfig` directory, allowing easy dependency
usage on the SDK libraries through standard pkg_config model.

For Rust, the Cargo repository that old all the Camelot crate that can be used by any library or application are stored in the
`usr/local/share/cargo/registry/camelot_sdk` directory.


This repository can be used as a template to generate a SDK for any other board that hold a Camelot-supported SoC such as
stm32l429, stm32wb55, stm32f429 and so on. Use the Sentry kernel examples configurations and device-tree for each of them as input
to initiate your board configuration and device-tree configuration.
