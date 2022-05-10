# 00001 - Payment

- id: 1
- name: `Payment`
- [Definition file](https://github.com/flare-foundation/attestation-client/blob/main/lib/verification/attestation-types/t-00001-payment.ts)

## Description

The purpose of this attestation type is to provide a general attestation of a _native payment_  transaction. Native payment on a blockchain is an elementary payment with the system currency, where funds are sent from one address to another. In the case of chains based on the UTXO model, a specific generalization is applied (see Definitions in the [MCC docs](https://github.com/flare-foundation/multi-chain-client/tree/main/docs/definitions)).

A successful attestation is provided by extracting the following data about a transaction:

- Block number
- Block timestamp
- Transaction hash
- Selected transaction input index (in UTXO chains)
- Selected transaction output index (in UTXO chains)
- Source address
- Receiving address
- Spent amount
- Received amount
- [Payment reference](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/payment-reference.md)
- Whether transaction is sending from a single address to a single other address
- [Transaction status](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/transaction-status.md)

Due to technical limitations on UTXO chains, the procedure for the attestation differs according to the existence of a standardized payment reference. In essence, payments with a standardized payment references undergo [full attestation](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/account-based-vs-utxo-chains.md) whereas other payments undergo partial attestation. In non-UTXO chains full attestation is always performed.

## Request format

Beside the standard fields (`attestationType`, `sourceId` and `upperBoundProof`) the request for `Payment` attestation type contains in addition fields `id`, `utxo` and `inUtxo`.

| Name              | Size (bytes) | Internal type      | Description                                                                  |
| ----------------- | ------------ | ------------------ | ---------------------------------------------------------------------------- |
| `attestationType` | 2            | `AttestationType`  | Attestation type id for this request, see `AttestationType` enum.            |
| `sourceId`        | 4            | `SourceId`         | The ID of the underlying chain, see `SourceId` enum.                         |
| `upperBoundProof` | 32           | `ByteSequenceLike` | The hash of the confirmation block for an upper query window boundary block. |
| `id`              | 32           | `ByteSequenceLike` | Transaction hash to search for.                                              |
| `inUtxo`          | 1            | `NumberLike`       | Index of the source address on UTXO chains. Always 0 on non-UTXO chains.     |
| `utxo`            | 1            | `NumberLike`       | Index of the receiving address on UTXO chains. Always 0 on non-UTXO chains.  |

## Verification rules

- The transaction being attested must be a [native payment](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/native-payment.md).
- Payment reference is calculated only if the attested transaction conforms to a [standardized payment reference](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/payment-reference.md). Otherwise, the payment reference is 0.
- Status of the attestation is determined as described [here](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/transaction-status.md).
- If the payment status is failure (1 or 2), the received amount should be 0 and the spent amount should be only the fee spent in case of non-UTXO chains. In case of UTXO chains 0 value is provided for spent amount.
- The values of `utxo` and `inUtxo` on non-UTXO chains must always be 0.

### UTXO (BTC, LTC, DOGE) chains

- Full attestation is performed only if the standardized payment reference exists (see [here](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/account-based-vs-utxo-chains.md) for details). Otherwise, partial attestation is performed.
- Source address exists only if there is a unique source address on the selected input (`inUtxo`). To determine it, one needs to make an additional RPC API call. If the source address does not exist, it is indicated by 0 in the response. The spent amount is 0 in this case. If the source address exists, a hash (sha3) is provided for `sourceAddressHash` in response.
- The receiving address may not exist on the selected output (`utxo`). In this case it is indicated in the response by 0. The received amount is 0 in this case. If the receiving address exists, its hash is provided for `receivingAddressHash` in response.

## Response format

| Name                   | Type         | Description                                                 |
| ---------------------- | ------------ | ----------------------------------------------------------- |
| `blockNumber`          | `uint64`     | Number of the transaction block on the underlying chain.    |
| `blockTimestamp`       | `uint64`     | Timestamp of the transaction block on the underlying chain. |
| `transactionHash`      | `bytes32`    | Hash of the transaction on the underlying chain.            |
| `inUtxo`               | `uint8`      | Index of the transaction input indicating source address on UTXO chains, 0 on non-UTXO chains. |
| `utxo`                 | `uint8`      | Output index for a transaction with multiple outputs on UTXO chains, 0 on non-UTXO chains. The same as in the `utxo` parameter from the request. |
| `sourceAddressHash`    | `bytes32`    | Hash of the source address viewed as a string (the one indicated by the `inUtxo` parameter for UTXO blockchains). |
| `receivingAddressHash` | `bytes32`    | Hash of the receiving address as a string (the one indicated by the `utxo` parameter for UTXO blockchains). |
| `spentAmount`          | `int256`     | The amount that went out of the source address, in the smallest underlying units. In non-UTXO chains it includes both payment value and fee (gas). Calculation for UTXO chains depends on the existence of standardized payment reference. If it exists, it is calculated as `outgoing_amount - returned_amount` and can be negative. If the standardized payment reference does not exist, then it is just the spent amount on the input indicated by `inUtxo`. |
| `receivedAmount`       | `int256`     | The amount received by the receiving address, in the smallest underlying units. Can be negative in UTXO chains. |
| `paymentReference`     | `bytes32`    | Standardized payment reference, if it exists, 0 otherwise. |
| `oneToOne`             | `bool`       | `true` if the transaction has exactly one source address and exactly one receiving address (different from source). |
| `status`               | `uint8`      | Transaction success status, can have 3 values: **0** (Success), **1** (Failure due to sender: This is the default failure) and **2** (Failure due to receiver: Bad destination address). |

Next: [00002 - Balance Decreasing Transaction](./00002-balance-decreasing-transaction.md)

[Back to home](../README.md)
