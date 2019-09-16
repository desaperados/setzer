# Setzer

Publish signed prices to an Ethereum oracle.

<img width="1220" alt="Screen Shot 2019-09-16 at 9 10 43 AM" src="https://user-images.githubusercontent.com/5028/64927675-f1b35580-d861-11e9-8ade-27b694b87fd8.png">


## Commands

```
Usage: setzer [<opts>] <command> [<args>]
   or: setzer <command> --help

Commands:

   broadcast       Broadcast signed prices over ssb
   deploy          Deploy a new oracle contract
   help            Print help about setzer or one of its subcommands
   latest          Print most recent quorum price messages for a pair
   pairs           List all supported price pairs
   poke            Poke an on-chain oracle with latest prices
   price           Fetch exchange price(s) for a given asset or pair
   sources         Show price sources for a given asset or pair
   test            Test all price sources
```

## Basic usage

__setzer deploy__ `[--quorum] <pair>`

* Deploys the MakerDAO [medianizer](https://github.com/makerdao/median/blob/master/src/median.sol) contract with the `toll` modifier removed to allow anyone to read prices from the oracle.
* The initial quorum (`bar`) required to push new prices to the contract is set to 1 by default or `--quorum=<number>` if provided.

__setzer broadcast__ `[--interval] [--follow] [<pair>]`

* Continuously broadcasts prices over the setzer ssb network at a given `interval`.
* Prices are signed with an Ethereum key, allowing message authenticity to be verified.
* `pair` can be singular e.g `ethusd` or a space-separated list of pairs. If no pairs are specified, all `setzer pairs` will be used.
* `pair` is assumed to ba a USD pair and can be upper or lowercase, with or without quote appended e.g ETH or ETHUSD.
* Broadcast will run in the background unless `--follow` is specified.

__setzer poke__ ``<address>``

* Pokes an on-chain oracle at a given address.
* Retrieves the `pair`, `quorum`, and `age` from the oracle and queries the ssb network for valid price data. If sufficient fresh price data is available from feeds trusted by the oracle, a `poke` transaction is submitted to update the on-chain price.

## Demo

```bash
# Start an Ethereum testnet
$ dapp testnet

# In a new shell
$ export ETH_RPC_URL=http://127.0.0.1:8545
$ export ETH_KEYSTORE=$HOME/.dapp/testnet/8545/keystore
$ export ETH_PASSWORD=/dev/null
$ export ETH_FROM=$(seth accounts ls | sed 1q | awk '{print substr ($0, 0, 42)}')

# Deploy an oracle to testnet and trust the deployer
$ ORACLE=$(setzer deploy ETHUSD --quorum 1)
$ seth send $ORACLE 'lift(address)' $ETH_FROM

# Broadcast signed prices
$ setzer broadcast --interval 120 ETHUSD
2019-09-16 07:44:03 ETHUSD 188.9850000000
2019-09-16 07:46:15 ETHUSD 188.9541217800
2019-09-16 07:48:28 ETHUSD 188.9662400800
2019-09-16 07:50:41 ETHUSD 189.4291848000
2019-09-16 07:52:55 ETHUSD 189.3350000000
2019-09-16 07:55:06 ETHUSD 189.4150000000
2019-09-16 07:57:21 ETHUSD 189.5800000000

# Update the testnet oracle with latest prices
$ setzer poke $ORACLE
seth send 0x6816ae807feFbBe49e80331C069Ae4BA7Dd6be3b 'poke(uint256[] memory,uint256[] memory,uint8[] memory,bytes32[] memory,bytes32[] memory)' '[00000000000000000000000000000000000000000000000a1ddf6d3eaf660000]' '[000000000000000000000000000000000000000000000000000000005d7db476]' '[000000000000000000000000000000000000000000000000000000000000001c]' '[e129727276e098cc91e069f6682ae55378255ef5e69733b984007e0fa5a1c415]' '[2d8316fca04337037fa723788603c87ab8c25bce8d8809b79614a14e0a207f6f]'
seth-send: Published transaction with 484 bytes of calldata.
seth-send: Waiting for transaction receipt...
seth-send: Transaction included in block 81.
```

## Price sources

List price sources for any given pair with `sources`

```bash
$ setzer sources btcusd
binance
bitfinex
bitstamp
coinbase
gemini
kraken
upbit
```

Specify the `source` to get a price from a particular exchange. Without a `source` all sources are queried and the median is returned. __Broadcast publishes the median price__.
```bash
$ setzer price btc binance
10368.1388898810

$ setzer price btc
10304.4800000000
```


## Not addressed

* Price signer governance
* Rewards
* ssb invites

## Installation

Dependencies:

* GNU [bc](https://www.gnu.org/software/bc/)
* [curl](https://curl.haxx.se/download.html)
* [dapp](https://github.com/dapphub/dapptools/tree/master/src/dapp)
* GNU [datamash](https://www.gnu.org/software/datamash/)
* GNU `date`
* [jshon](http://kmkeen.com/jshon/)
* [Jq](https://stedolan.github.io/jq/)
* GNU `timeout`
* [ssb](https://github.com/ssbc/ssb-server)

Install via make:

* `make link` -  link setzer into `/usr/local`
* `make install` -  copy setzer into `/usr/local`
* `make uninstall` -  remove setzer from `/usr/local`

## Configuration

* `SETZER_DIR` - Tmp cache directory (default: ~/.setzer)
* `SETZER_CACHE_EXPIRY` - Price fetch cache expiry (default: 60) sec.
* `SETZER_TIMEOUT` - HTTP request timeout (default: 10) sec.
* `SETZER_BROADCAST_AGE` - Max. price broadcast validity (default: 900) 15 min.
* `SETZER_BROADCAST_INTERVAL` Price broadcast interval (default: 120) 2 min.
* `SETZER_QUORUM` Min. signed prices required to update an oracle

## Todo

* Set poke quorum via oracle
* Specify asset pairs via config file
* Poke bot
* Nix | dapp.tools install
* Remove jshon
