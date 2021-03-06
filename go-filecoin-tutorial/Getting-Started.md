# Getting started

This is a step-by-step guide for installing and running a Filecoin node connected to the testnet on your local machine. Subsequent tutorials explain how to [mine Filecoin](Mining-Filecoin) or [store data](Storing-on-Filecoin) with the node.

## Table of contents

* [System requirements](#system-requirements)
* [Installing dependencies and system configuration](#installing-dependencies-and-system-configuration)
* [Building Filecoin and running tests](#building-filecoin-and-running-tests)
* [Start running Filecoin](#start-running-filecoin)
* [Get FIL from the Filecoin faucet](#get-fil-from-the-filecoin-faucet)
* [Wait for chain sync](#wait-for-chain-sync)
* [Viewing network information](#viewing-network-information)

### System requirements

Go-filecoin can build and run on most GNU/Linux and MacOS systems. Windows is not yet supported.

A validating node can run on most systems with **at least 8GB of RAM**. Mining nodes in particular require significant RAM and GPU resources, depending on the sector configuration being implemented.

### Installing dependencies and system configuration

Clone the `go-filecoin` git repository and enter it:

   ```sh
    mkdir -p /path/to/filecoin-project
    git clone https://github.com/filecoin-project/go-filecoin.git /path/to/filecoin-project/go-filecoin
   ```

#### Installing Go

The build process for `go-filecoin` requires [Go](https://golang.org/doc/install) >= v1.13.

> Installing Go for the first time? We recommend [this tutorial](https://www.ardanlabs.com/blog/2016/05/installing-go-and-your-workspace.html) which includes environment setup.

Due to the use of `cgo` in `go-filecoin`, a C compiler is required to build it whether a prebuilt library is being used or it is compiled from source. To use `gcc` (e.g. `export CC=gcc`), v7.4.0 or higher is required.

The build process will download a static library containing the [Filecoin proofs implementation](https://github.com/filecoin-project/rust-fil-proofs) (which is written in Rust).

> **NOTICE:** To build proofs from source, (1) a Rust development environment must be installed and (2) the environment variable `FFI_BUILD_FROM_SOURCE=1` must be set. More information can be found in [filecoin-ffi](https://github.com/filecoin-project/filecoin-ffi).

#### Installing dependencies

1. Load all the Git submodules:

    ```sh
    git submodule update --init --recursive
    ```

2. Initialize the build dependencies:

    ```sh
    make deps
    ```

 > **NOTICE:** The first `deps` start up can be **slow**, as very large parameter files are either downloaded or generated locally in `/var/tmp/filecoin-proof-parameters`. Have patience; future runs will be faster.

### Building Filecoin and running tests

1. Build the binary:
    ```sh
    make
    ```

2. Run the unit tests:
    ```sh
    go run ./build test
    ```

3. Optionally, building and tests can be combined:
    ```sh
    go run ./build best
    ```

Other handy build commands include:

```sh
# Check the code for style and correctness issues
go run ./build lint

# Run different categories of tests by toggling their flags
go run ./build test -unit=false -integration=true -functional=true

# Test with a coverage report
go run ./build test -cover

# Test with Go's race-condition instrumentation and warnings (see https://blog.golang.org/race-detector)
go run ./build test -race

# Deps, Lint, Build, Test (any args will be passed to `test`)
go run ./build all
```

> **NOTICE:** Any flag passed to `go run ./build test` (e.g. `-cover`) will be passed on to `go test`.

**For all problems with the build**, please consult the [Troubleshooting](https://go.filecoin.io/go-filecoin-tutorial/Troubleshooting-&-FAQ.html) section of this documentation.

## Start running Filecoin

1. If `go-filecoin` has been run on the system before, remove existing Filecoin repo (**this will delete all previous filecoin data**):
    ```sh
    rm -rf ~/.filecoin
    ```

2. Initialize go-filecoin:
    ```sh
    go-filecoin init --genesisfile=https://ipfs.io/ipfs/QmXZQeezX1x8uRQX9EUaYxnyivUpTfJqQTvszk3c8SnFPN/testnet.car --network=testnet
    ```

3. Start the go-filecoin daemon:
    ```sh
    go-filecoin daemon
    ```
    
This should return "My peer ID is `<peerID>`", where `<peerID>` is a long [CID](https://github.com/filecoin-project/specs/blob/master/definitions.md#cid) string starting with "Qm".

4. Print a list of bootstrap node addresses:
    ```sh
    go-filecoin config bootstrap.addresses
    ```

    
5. Choose any address from the list you just printed, and connect to it (Automatic peer discovery and connection coming soon.):
    ```sh
    go-filecoin swarm connect <any-filecoin-node-mulitaddr>
    ```
    
 > **NOTICE:** This can be **slow** the first time. The filecoin node needs a large parameter file for proofs, stored in `/tmp/filecoin-proof-parameters`. It is usually generated by the `deps` build step. If these files are missing they will be regenerated, which can take up to an hour. We are working on a better solution.

6. Check the node's connectivity:
    ```sh
    go-filecoin swarm peers                  # lists addresses of peers to which you're connected
    ```

7. The last segment of a peer's address is its `peerID`. Test the connection directly to a peer with:
    
    ```sh
    go-filecoin ping <peerID>  # Pings the peer and displays round-trip latency.
    ```

The node should now be connected to some peers and will begin downloading and validating the blockchain.

🎉 Woohoo! You are now running a Filecoin node and connected to the network. This is the anatomy of a basic node:
![Diagram of a single node and its components](./images/getting-started-node-diagram.png)

 > **NOTICE:** The daemon is now running indefinitely in its own Terminal (`Ctrl + C` to quit). To run other `go-filecoin` commands, open a second Terminal tab or window (`Cmd + T` on Mac)._

_Need help? See [Troubleshooting & FAQ](Troubleshooting-&-FAQ) or [#fil-dev on Matrix chat](https://riot.im/app/#/room/#fil-dev:matrix.org)._


## Get FIL from the Filecoin faucet

**Once your chain has finished syncing**, you will be able to use the faucet to get filecoin tokens (FIL). Before Filecoin nodes can participate in the marketplace, they will need some start-up FIL. Clients need FIL in their accounts to make storage deals with miners. Miners use FIL for collateral when initially pledging storage to the network.

During early testing, mock FIL can be obtained from the Filecoin faucet. The "faucet" is thusly named because it drips (or pours) FIL into those who stick their wallets under it. Using mock FIL allows for preliminary testing of market dynamics without the requirement for any money to actually change hands.

All balances of FIL are stored in wallets. When a node is newly created, it will have a Filecoin wallet with a balance of 0 FIL.

1. Retrieve your wallet address:
    ```sh
    go-filecoin address ls
   ```
    
2. The output should be a long alphanumeric string. Go to the testnet faucet at [https://faucet.testnet.filecoin.io] and submit that wallet address. It will take a minute for the FIL to arrive in the wallet.

    * Alternatively, you can tap the faucet from the command line:
        ```sh
        export WALLET_ADDR=`go-filecoin address ls`    # fetch your wallet address into a handy variable
        MESSAGE_CID=`curl -X POST -F "address=${WALLET_ADDR}" "https://faucet.testnet.filecoin.io/send"`
        ```
        
3. The faucet will provide a message CID. If thr chain is already synced with the network, this message should be processed in about 30 seconds. The following command can be run in order to wait for confirmation:

    ```sh
    go-filecoin message wait ${MESSAGE_CID}
    ```

4. Verify that the FIL has landed in the wallet by checking the wallet balance:
    ```sh
    go-filecoin wallet balance ${WALLET_ADDR}
    ```
    
## Wait for chain sync
🎉 Congratulations, you're now connected to Filecoin! The daemon is now busy syncing and validating the existing blockchain, which can take awhile -- hours or even days depending on network age and activity.

During this initial sync time ther will be intense activity on one CPU core. Find out what the current block height is first by visiting the [network stats page](https://stats.testnet.filecoin.io) then observe the nodes syncing progress:
```sh
watch -n 10 'go-filecoin show block $(go-filecoin chain head | head -n 1)'
# Mac users will need to install watch first: brew install watch
````

After syncing is complete, you can begin mining or storing data on the Filecoin network:
- [Mining Filecoin](Mining-Filecoin)
- [Storing Data](Storing-on-Filecoin)

## Viewing network information

There are a few visualisation tools to help users understand what is happening within the Filecoin network, such as the official [network stats](http://stats.testnet.filecoin.io/) page as well as the community-managed block explorers [filscan.io](https://filscan.io) and [filscout.io](https://filscout.io).
