The following methods must be implemented as JSON-RPC over HTTP. See an [example implementation](https://github.com/airswap/airswap-maker-kit-examples/blob/master/api/handlers.js#L217).

## `getSenderSideOrder`

Given a `signerAmount`, `senderWallet`, and token pair, return a complete order. The `senderAmount` value is the amount the taker would send. The taker is **buying** from you.

**Example Request**

```json
{
  "jsonrpc": "2.0",
  "method": "getSenderSideOrder",
  "params": {
    "signerAmount": "10000",
    "signerToken": "0x27054b13b1b798b345b591a4d22e6562d47ea75a",
    "senderToken": "0xc778417e063141139fce010982780140aa0cd5ab",
    "senderWallet": "0x1FF808E34E4DF60326a3fc4c2b0F80748A3D60c2"
  },
  "id": "123"
}
```

| Param          | Type      | Description                                 |
| :------------- | :-------- | :------------------------------------------ |
| `signerAmount` | `uint256` | Amount of ERC-20 the signer would transfer. |
| `signerToken`  | `address` | Token the signer would transfer.            |
| `senderToken`  | `address` | Token the sender would transfer.            |
| `senderWallet` | `address` | Wallet of the sender.                       |

A successful `getSenderSideOrder` returns a signed [Order](./orders-and-signatures.md#creating-orders) object including the requested `senderAmount`.

## `getSignerSideOrder`

Given a `senderAmount`, `senderWallet`, and token pair, return a complete order. The `signerAmount` value is the amount you would send. The taker is **selling** to you.

**Example Request**

```json
{
  "jsonrpc": "2.0",
  "method": "getSignerSideOrder",
  "params": {
    "signerToken": "0x27054b13b1b798b345b591a4d22e6562d47ea75a",
    "senderWallet": "0x1FF808E34E4DF60326a3fc4c2b0F80748A3D60c2",
    "senderToken": "0xc778417e063141139fce010982780140aa0cd5ab",
    "senderAmount": "100000000"
  },
  "id": "123"
}
```

| Param          | Type      | Description                                     |
| :------------- | :-------- | :---------------------------------------------- |
| `senderAmount` | `uint256` | The amount of ERC-20 the sender would transfer. |
| `signerToken`  | `address` | The token the signer would transfer.            |
| `senderToken`  | `address` | The token the sender would transfer.            |
| `senderWallet` | `address` | The wallet of the sender.                       |

A successful `getSignerSideOrder` returns a signed [Order](./orders-and-signatures.md#creating-orders) object including the requested `signerAmount`.

# Creating Orders

An AirSwap `Order` has the following properties.

| Param     | Type        | Description                                             |
| :-------- | :---------- | :------------------------------------------------------ |
| nonce     | `uint256`   | Unique per signer and should be sequential              |
| expiry    | `uint256`   | Expiry in seconds since 1 January 1970                  |
| signer    | `Party`     | Party to the trade that sets and signs terms            |
| sender    | `Party`     | Party to the trade that accepts and sends terms         |
| affiliate | `Party`     | `optional` Party compensated for facilitating the trade |
| signature | `Signature` | Signature of the order, described below                 |

Each `Party` has the following properties.

| Param  | Type      | Description                                     |
| :----- | :-------- | :---------------------------------------------- |
| kind   | `bytes4`  | `0x36372b07` (ERC-20) or `0x80ac58cd` (ERC-721) |
| wallet | `address` | Wallet address of the party                     |
| token  | `address` | Contract address of the token                   |
| amount | `uint256` | Amount of the token (ERC-20)                    |
| id     | `uint256` | ID of the token (ERC-721)                       |

These values correlate to the structs in [Types](../contracts/types.md).

## Example

```json
{
  "nonce": "100",
  "expiry": "1566941284",
  "signer": {
    "kind": "0x36372b07",
    "wallet": "0x6556b252b05ad2ff5435d04a812b77875fa2bdbe",
    "token": "0x27054b13b1b798b345b591a4d22e6562d47ea75a",
    "amount": "10000",
    "id": "0"
  },
  "sender": {
    "kind": "0x36372b07",
    "wallet": "0x1FF808E34E4DF60326a3fc4c2b0F80748A3D60c2",
    "token": "0xc778417e063141139fce010982780140aa0cd5ab",
    "amount": "100000000",
    "id": "0"
  },
  "affiliate": {
    "kind": "0x36372b07",
    "wallet": "0x0000000000000000000000000000000000000000",
    "token": "0x0000000000000000000000000000000000000000",
    "amount": "0",
    "id": "0"
  },
  "signature": {
    "signatory": "0x6556b252b05ad2ff5435d04a812b77875fa2bdbe",
    "validator": "0x3E0c31C3D4067Ed5d7d294F08B79B6003B7bf9c8",
    "version": "0x45",
    "v": "28",
    "r": "0x589bb063fc85f49ad096ec9513c45b3e93f5a2da4efe0706db9a2b755121f4c2",
    "s": "0x73075fbae37e5a4954a6e57e0c056d130b582ce390b56fd69f0bb2e103d07e70"
  }
}
```

The `@airswap/order-utils` package includes an `orders.getOrder` function that can be used to create new orders with correct default values.