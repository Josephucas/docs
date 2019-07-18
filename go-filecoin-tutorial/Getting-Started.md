# Getting Started

This is a step-by-step guide for installing and running a Filecoin node connected to the [User&nbsp;Devnet](Devnets#user) on your computer. Subsequent tutorials explain how to [mine Filecoin](Mining-Filecoin) or [store data](Storing-on-Filecoin) with your node.

## Table of Contents

* [Install Filecoin and its dependencies](#install-filecoin-and-its-dependencies)
* [Start running Filecoin](#start-running-filecoin)
* [Name your Filecoin node](#name-your-filecoin-node)
* [Start streaming activity from your node](#start-streaming-activity-from-your-node)
* [Get FIL from the Filecoin faucet](#get-fil-from-the-filecoin-faucet)
* [Wait for chain sync](#wait-for-chain-sync)

## Install Filecoin (and its dependencies)

<!--
We have two installation methods available:
* install from binary (recommended for most)
* install from source (requires golang installation, rust and other tools)

### Installing from binary
Coming soon.
-->
### Installing from binary
  - Go to the latest release for [go-filecoin Releases page on GitHub](https://github.com/filecoin-project/go-filecoin/releases/latest). 
  - Click on the `.tar.gz` link that matches your operating system (OSX or Linux) 
  - Unzip the downloaded file
  - Fire up your terminal (_Terminal.app_ on MacOS) and `cd` into your newly created `filecoin` directory.
  - Create a directory for proofs parameters and fetch them via `paramfetch`:
    ```sh
    mkdir -p /tmp/filecoin-proof-parameters
    ./paramfetch -a -v
    # be warned, this can take a long time
    ```
  - Add `go-filecoin` to your path by opening the `filecoin` folder inside your Terminal and running:
    ```sh
    export PATH="$(pwd)":$PATH
    ```
  - You will need to add the previous line to your shell init file like `~/.bash_profile` (advanced) or run it from the `filecoin` directory in each terminal you open.

### Installing from source

Use these steps to install filecoin:

1. Build dependencies
   - golang `1.12.1`
   - rust/rustup ([follow the instructions on rust-fil-proofs](https://github.com/filecoin-project/rust-fil-proofs#install-and-configure-rust), **DO NOT** install from homebrew)
   - pkg-config
   - jq

1. Find the latest git tag for the user devnet from the project [README](https://github.com/filecoin-project/go-filecoin#filecoin-go-filecoin) by clicking the badge (user devnet etc) that takes you to the release page (e.g., "0.3.2").

1. Ensure you have the correct code for joining the user devnet:
    ```sh
    cd $GOPATH/src/github.com/filecoin-project/go-filecoin
    git fetch origin
    git checkout $USER_DEVNET_TAG
    ```

1. Install dependencies:
    ```sh
    FILECOIN_USE_PRECOMPILED_RUST_PROOFS=true go run ./build deps
    ```

1. Build the project:
    ```sh
    go run ./build build
    cp go-filecoin $GOPATH/bin
    ```

_If you run into an error: `panic: json: cannot unmarshal string into Go struct field .Power of type uint64` you will need to remove the `gen.json` file from the fixtures._

```sh
rm -f ./fixtures/{test,live}/gen.json
```

More info: Issue [#2859](https://github.com/filecoin-project/go-filecoin/issues/2859#issuecomment-497402147)

## Start running Filecoin

1. If you have run go-filecoin before, remove existing Filecoin repo (**this will delete all previous filecoin data**):
    ```sh
    rm -rf ~/.filecoin
    ```

1. Initialize go-filecoin. The `--devnet-user` flag connects you to our main devnet:
    ```sh
    go-filecoin init --devnet-user --genesisfile=https://genesis.user.kittyhawk.wtf/genesis.car
    ```

1. Start your go-filecoin daemon:
    ```sh
    go-filecoin daemon
    ```
    This should return "My peer ID is `<peerID>`", where `<peerID>` is a long [CID](https://github.com/filecoin-project/specs/blob/master/definitions.md#cid) string starting with "Qm".

    Note: this can be **slow** the first time. The filecoin node needs a large parameter file for proofs, stored in `/tmp/filecoin-proof-parameters`. It's usually generated by the `deps` build step. If these files are missing they will be regenerated, which can take up to an hour. We are working on a better solution.

1. Check your connectivity:
    ```sh
    go-filecoin swarm peers                  # lists addresses of peers to which you're connected
    ```
    The last segment of a peer's address is its `peerID`. Test your connection directly to a peer with:
    ```sh
    go-filecoin ping <peerID>                # Pings the peer and displays round-trip latency.
    ```

🎉 Woohoo! You are now running a Filecoin node and connected to the network. This is the anatomy of a basic node:
![Diagram of a single node and its components](./images/getting-started-node-diagram.png)

_Note: The daemon is now running indefinitely in its own Terminal (`Ctrl + C` to quit). To run other `go-filecoin` commands, you'll need to open a second Terminal tab or window (`Cmd + T` on Mac)._

_Need help? See [Troubleshooting & FAQ](Troubleshooting-&-FAQ) or [#fil-dev on Matrix chat](https://riot.im/app/#/room/#fil-dev:matrix.org)._

## Name your Filecoin node

By default, nodes are referenced by long, alphanumeric node IDs. You can give your node a human-readable nickname.
* Nicknames can only contain letter characters (no numbers, spaces, or other special characters).

1. Open a new Terminal window and set your node nickname (replace `Pizzanode` with the name of your choice): 
    ```sh
    go-filecoin config heartbeat.nickname "Pizzanode"
    ````
1. The new name takes effect immediately, no need to restart. You can check the configured name with:
    ```sh
    go-filecoin config heartbeat.nickname
    ````

## Start streaming activity from your node

We have a few visualization tools to understand what's happening on the Filecoin network: the [Network Stats](https://stats.kittyhawk.wtf/) and [block explorer](Block-Explorer). 

To see your node on the network stats, you'll need to opt in to streaming your node's logs. Open a new Terminal window and run:
```sh
go-filecoin config heartbeat.beatTarget "/dns4/stats-infra.kittyhawk.wtf/tcp/8080/ipfs/QmUWmZnpZb6xFryNDeNU7KcJ1Af5oHy7fB9npU67sseEjR"
```
Restart the currently running `go-filecoin daemon` process and then go to the [Network Stats](https://stats.kittyhawk.wtf/) and watch your node reach consensus with the rest of the network.

## Wait for chain sync
🎉 Congrats, you're now connected to Filecoin! Your daemon is now busy syncing and validating the existing blockchain, which can take awhile -- hours or even days depending on network age and activity.

During this time, you'll observe intense activity on one CPU core. Find out what the current block height is first by visiting the [Network Stats Page](https://stats.kittyhawk.wtf/). Then you can watch your node's syncing progress:
````sh
watch -n 10 'go-filecoin show block $(go-filecoin chain head | head -n 1)'
# Mac users will need to install watch first: brew install watch
````

## Get FIL from the Filecoin faucet

**Once your chain has finished syncing**, you will be able to use the faucet to get filecoin tokens (FIL). Before Filecoin nodes can participate in the marketplace, they will need some start-up FIL. Clients need FIL in their accounts to make storage deals with miners. Miners use FIL for collateral when initially pledging storage to the network.

During early testing, you can obtain mock FIL from the Filecoin faucet. The "faucet" is thusly named because it drips (or pours) FIL into those who stick their wallets under it. Using mock FIL allows for preliminary testing of market dynamics without the requirement for any money to actually change hands.

All balances of FIL are stored in wallets. When a node is newly created, it will have a Filecoin wallet with a balance of 0 FIL.
1. Let's get you some FIL. Get your wallet address:
    ```sh
    go-filecoin address ls
    ```
1. The output should be a long alphanumeric string. Go to the User Devnet faucet at [http://user.kittyhawk.wtf:9797](http://user.kittyhawk.wtf:9797) and submit that wallet address. It will take a minute for the FIL to land in your wallet.

    1. Alternatively, you can tap the faucet from the command line:
        ```sh
        export WALLET_ADDR=`go-filecoin address ls`    # fetch your wallet address into a handy variable
        MESSAGE_CID=`curl -X POST -F "target=${WALLET_ADDR}" "http://user.kittyhawk.wtf:9797/tap" | cut -d" " -f4`
        ```
1. The faucet will provide you with a message CID. If your chain is already synced with the network, this message should be processed in about 30 seconds. You can run the following command to wait for confirmation:

    ```sh
    go-filecoin message wait ${MESSAGE_CID}
    ```

1. To verify that the FIL has landed in your wallet, check your wallet balance:
    ```sh
    go-filecoin wallet balance ${WALLET_ADDR}
    ```

After syncing is complete, you can begin mining or storing data on the Filecoin network! Choose your adventure: 
- [Mining Filecoin](Mining-Filecoin)
- [Storing Data](Storing-on-Filecoin)