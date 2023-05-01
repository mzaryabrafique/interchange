# Interchange module Rollup on Celestia Network
**interchange** is a blockchain built using Cosmos SDK and Tendermint and created with [Ignite CLI](https://ignite.com/cli).

Ignite CLI is a command line interface that allows users to quickly and easily create blockchain networks. By using Ignite CLI, we can quickly create a new blockchain without having to manually set up all the necessary components.

The module has order books, buy orders, and sell orders.

- First, create an order book for a pair of token.
- After an order book exists, you can create buy and sell orders for this pair of token.
The module uses the Inter-Blockchain Communication protocol IBC. By using IBC, the module can create order books so that multiple blockchains can interact and exchange their token.

We create an order book pair with a token from one blockchain and another token from another blockchain. Here, we call the module we create the `dex` module.

## Working of the module
When a user exchanges a token with the `dex` module, a `voucher` of that token is received on the other blockchain. This `voucher` is similar to how an ibc-transfer is constructed. Since a blockchain module does not have the rights to mint new token of a blockchain into existence, the token on the target chain is locked up, and the buyer receives a `voucher` of that token.

This process can be reversed when the voucher gets burned to unlock the original token.

## Assumptions of the design:

An order book can be created for the exchange of any tokens between any pair of chains.

- Both blockchains require the dex module to be installed and running.
- There can only be one order book for a pair of token at the same time.
- A specific chain cannot mint new coins of its native token.

This module is inspired by the `ibc-transfer` module on the Cosmos SDK. The `dex` module we created has similarities, like the `voucher` creation.

However, the new dex module we created is more complex because it supports creation of:

- Several types of packets to send
- Several types of acknowledgments to treat
- More complex logic on how to treat a packet on receipt, on timeout, and more

## Features
The features supported by the interchain exchange module are:

- Create an exchange order book for a token pair between two chains
- Send sell orders on source chain
- Send buy orders on target chain
- Cancel sell or buy orders

## Setup

```
ignite chain build
chmod +x init-local.sh
./init-local.sh
```
