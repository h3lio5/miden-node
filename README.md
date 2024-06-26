# Miden node

[![LICENSE](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/0xPolygonMiden/miden-node/blob/main/LICENSE)
[![test](https://github.com/0xPolygonMiden/miden-node/actions/workflows/test.yml/badge.svg)](https://github.com/0xPolygonMiden/miden-node/actions/workflows/test.yml)
[![RUST_VERSION](https://img.shields.io/badge/rustc-1.78+-lightgray.svg)](https://www.rust-lang.org/tools/install)
[![crates.io](https://img.shields.io/crates/v/miden-node)](https://crates.io/crates/miden-node)

This repository holds the Miden node; that is, the software which processes transactions and creates blocks for the Miden rollup.

### Status

The Miden node is still under heavy development and the project can be considered to be in an _alpha_ stage. Many features are yet to be implemented and there is a number of limitations which we will lift in the near future.

At this point, we are developing the Miden node for a centralized operator. Thus, the work does not yet include such components as P2P networking and consensus. These will also be added in the future.

## Architecture

The Miden node is made up of 3 main components, which communicate over gRPC:

- **[RPC](crates/rpc):** an externally-facing component through which clients can interact with the node. It receives client requests (e.g., to synchronize with the latest state of the chain, or to submit transactions), performs basic validation, and forwards the requests to the appropriate internal components.
- **[Store](crates/store):** maintains the state of the chain. It serves as the "source of truth" for the chain - i.e., if it is not in the store, the node does not consider it to be part of the chain.
- **[Block Producer](crates/block-producer):** accepts transactions from the RPC component, creates blocks containing those transactions, and sends them to the store.

All 3 components can either run as one process, or each component can run in its own process. See the [Running the node](#running-the-node) section for more details.

The diagram below illustrates high-level design of each component as well as basic interactions between them (components in light-grey are yet to be built).

![Architecture diagram](./assets/architecture.png)

## Usage

Before you can build and run the Miden node or any of its components, you'll need to make sure you have Rust [installed](https://www.rust-lang.org/tools/install). Miden node v0.2 requires Rust version **1.78** or later.

Depending on the platform, you may need to install additional libraries. For example, on Ubuntu 22.04 the following command ensures that all required libraries are installed.

```sh
sudo apt install llvm clang bindgen pkg-config libssl-dev libsqlite3-dev
```

### Installing the node

To install for production use cases, run:

```sh
cargo install --path bin/node
```

This will install the executable `miden-node` in your PATH, at `~/.cargo/bin/miden-node`.

Otherwise, if only to try the node out for testing, run:

```sh
cargo install --features testing --path bin/node
```

Currently, the only difference between the two is how long the `make-genesis` command will take to run (see next subsection).

### Generating the genesis file

Before running the node, you must first generate the genesis file. The contents of the genesis file are fully configurable through a genesis inputs file written in TOML. An example genesis inputs file can be found here: [genesis.toml](./config/genesis.toml)

To generate the genesis file, run:

```sh
miden-node make-genesis --inputs-path <path-to-config-file>
```

By default this will generate 1 file and 1 folder in the current directory:

- `genesis.dat`: the genesis file.
- `accounts` directory containing `.mac` files (one per account) for the accounts defined in the genesis inputs file. Each `.mac` file contains full serialization of an account, including code, storage, and authentication info.

### Running the node

Create a configuration file based on [node/miden-node.toml](./config/miden-node.toml), then create the necessary directories and start the node:

```sh
mkdir -p /opt/miden
miden-node start --config <path-to-config-file> <component-to-be-started>
```

Note that the `store.genesis_filepath` field in the config file must point to the `genesis.dat` file that you generated in the previous step.

### Running the node as separate components

If you intend on running the node as different processes, you will need to install and run each component separately.
Please, refer to each component's documentation:

- [RPC](crates/rpc/README.md#usage)
- [Store](crates/store/README.md#usage)
- [Block Producer](crates/block-producer/README.md#usage)

Each directory containing the executables also contains an example configuration file. Make sure that the configuration files are mutually consistent. That is, make sure that the URLs are valid and point to the right endpoint.

### Running the node using Docker

If you intend on running the node inside a Docker container, you will need to follow these steps:

1. Build the docker image from source

   ```sh
   cargo make docker-build-node
   ```

   This command will build the docker image for the Miden node and save it locally.

2. Run the Docker container

   ```sh
   # Using cargo-make
   cargo make docker-run-node

   # Manually
   docker run --name miden-node -p 57291:57291 -d miden-node-image
   ```

   This command will run the node as a container named `miden-node` using the `miden-node-image` and make port `57291` available (rpc endpoint).

3. Monitor container

   ```sh
   docker ps
   ```

    After running this command you should see the name of the container `miden-node` being outputted and marked as `Up`.

### Debian Packages

The debian packages allow for easy install for miden on debian based systems. Note that there are checksums available for the package.
Current support is for amd64, arm64 support coming soon.

To install the debian package:

```sh
sudo dpkg -i $package_name.deb
```

Note, when using the debian package to run the `make-genesis` function, you should define the location of your output:

```sh
miden-node make-genesis -i $input_location_for_genesis.toml -o $output_for_genesis.dat_and_accounts
```

The debian package has a checksum, you can verify this checksum by downloading the debian package and checksum file to the same directory and running the following command:

```sh
sha256sum --check $checksumfile
```

Please make sure you have the sha256sum program installed, for most linux operating systems this is already installed. If you wish to install it on your macOS, you can use brew:

```sh
brew install coreutils
```

## Testing

In order to test the node run the following command:

```sh
cargo-make test-all
```

## License

This project is [MIT licensed](./LICENSE).
