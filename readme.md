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

## Start Chains
To start using the interchain exchange, you will need to start two separate blockchains.
```
ignite chain serve -c mars.yml
```
To start the venus blockchain, you would run a similar command, but with the path to the venus.yml configuration file:
```
ignite chain serve -c venus.yml
```

# Setup Relayer
Set up an IBC relayer between two chains. If you have used a relayer in the past, reset the relayer configuration directory:
```
rm -rf ~/.ignite/relayer
```
Use the `ignite relayer configure` command:
```
ignite relayer configure -a --source-rpc "http://0.0.0.0:26657" --source-faucet "http://0.0.0.0:4500" --source-port "dex" --source-version "dex-1" --source-gasprice "0.0000025stake" --source-prefix "cosmos" --source-gaslimit 300000 --target-rpc "http://0.0.0.0:26659" --target-faucet "http://0.0.0.0:4501" --target-port "dex" --target-version "dex-1" --target-gasprice "0.0000025stake" --target-prefix "cosmos" --target-gaslimit 300000
```
To create a connection between the two chains, you can use the ignite relayer connect command:
```
ignite relayer connect
```

## Usage of Instructions
1. To create an order book for a pair of tokens
```
interchanged tx dex send-create-pair dex channel-0 marscoin venuscoin --from alice --chain-id mars --home ~/.mars
```
2. On the Mars blockchain, the `send-create-pair` command creates an empty sell order book to check run the following command
```
interchanged q dex list-sell-order-book
```
3. On the Venus blockchain, the same `send-createPair` command creates a buy order book:
```
interchanged q dex list-buy-order-book --node tcp://localhost:26659
```
4. After creating an order book, the next step is to create a sell order
```
interchanged tx dex send-sell-order dex channel-0 marscoin 10 venuscoin 15  --from alice --chain-id mars --home ~/.mars
```
5. `send-sell-order` command is used to create a sell order for 10 marscoin token and 15 venuscoin token, to check run the following queries
```
interchanged q bank balances $(interchanged keys show -a alice --home ~/.mars)
interchanged q dex list-sell-order-book
```
6. After creating a sell order, the next step in the trading process is typically to create a buy order
```
interchanged tx dex send-buy-order dex channel-0 marscoin 10 venuscoin 5 --from alice --chain-id venus --home ~/.venus --node tcp://localhost:26659
```
7. `send-buy-order` command is used to create a buy order for 10 marscoin token and 5 venuscoin token, to check run the following commands:
```
interchanged q bank balances $(interchanged keys show -a alice --home ~/.venus) --node tcp://localhost:26659
interchanged q dex list-buy-order-book --node tcp://localhost:26659
```
8. Perform an Exchange with a Sell Order (currently have two open orders for marscoin)
8.1 Send a sell order to the Mars chain using the following command:
```
interchanged tx dex send-sell-order dex channel-0 marscoin 5 venuscoin 3 --from alice --home ~/.mars
```
8.2 This will result in the amount of the buy order on the Venus chain being reduced by 5 marscoin, to check run the following commands:
```
interchanged q dex list-buy-order-book --node tcp://localhost:26659
interchanged q bank balances $(interchanged keys show -a alice --home ~/.mars)
interchanged q bank balances $(interchanged keys show -a alice --home ~/.venus) --node tcp://localhost:26659
```
9. Perform an Exchange with a Buy Order
```
interchanged tx dex send-buy-order dex channel-0 marscoin 5 venuscoin 15 --from alice --home ~/.venus --node tcp://localhost:26659
```
9.1 This buy order will be immediately filled on the Mars chain, and the creator of the sell order will receive 75 venuscoin vouchers as payment, to check run the following commands:
```
interchanged q bank balances $(interchanged keys show -a alice --home ~/.mars)
interchanged q dex list-sell-order-book
interchanged q bank balances $(interchanged keys show -a alice --home ~/.venus) --node tcp://localhost:26659
```
10. Complete Exchange with a Partially Filled Sell Order
```
interchanged tx dex send-sell-order dex channel-0 marscoin 10 venuscoin 3 --from alice --home ~/.mars
```
10.1 To check the balances, run the following commands:
```
interchanged q bank balances $(interchanged keys show -a alice --home ~/.venus) --node tcp://localhost:26659
interchanged q dex list-buy-order-book --node tcp://localhost:26659
interchanged q bank balances $(interchanged keys show -a alice --home ~/.mars)
interchanged q dex list-sell-order-book
```
11. Complete Exchange with a Partially Filled Buy Order
```
interchanged tx dex send-buy-order dex channel-0 marscoin 10 venuscoin 5 --from alice --home ~/.venus --node tcp://localhost:26659
```
11.1 To check, run the following commands:
```
interchanged q bank balances $(interchanged keys show -a alice --home ~/.mars)
interchanged q dex list-sell-order-book
interchanged q bank balances $(interchanged keys show -a alice --home ~/.venus) --node tcp://localhost:26659
interchanged q dex list-buy-order-book --node tcp://localhost:26659
```
12. Cancel an Order
```
interchanged tx dex cancel-sell-order dex channel-0 marscoin venuscoin 0 --from alice --home ~/.mars
```
12.1 To check, run the following commands:
```
interchanged q bank balances $(interchanged keys show -a alice --home ~/.mars) 
interchanged q dex list-sell-order-book
```
12.2 To cancel a buy order on the Venus chain, you can run the following command:
```
interchanged tx dex cancel-buy-order dex channel-0 marscoin venuscoin 1 --from alice --home ~/.venus --node tcp://localhost:26659
```
12.3 To check, run the following command:
```
interchanged q bank balances $(interchanged keys show -a alice --home ~/.venus) --node tcp://localhost:26659
interchanged q dex list-buy-order-book --node tcp://localhost:26659
```

Once the exchange is set up, users can send sell orders on the Mars chain and buy orders on the Venus chain. This allows them to offer their tokens for sale or purchase tokens from the exchange. In addition, users can also cancel their orders if needed.
