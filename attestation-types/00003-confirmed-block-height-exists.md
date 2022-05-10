# 00003 - Confirmed Block Height Exists

- id: 3
- name: `ConfirmedBlockHeightExists`
- [Definition file](https://github.com/flare-foundation/attestation-client/blob/main/lib/verification/attestation-types/t-00003-confirmed-block-height-exists.ts)

## Description

The purpose of this attestation type is to prove that a block on a certain height exists and it is confirmed.
The attestation uses `upperBoundProof` as a hash of the confirmation block for the highest confirmed block in the query.

A successful attestation is provided by providing the following data:

- Block number
- Block timestamp
- Number of confirmations used
- Average block production time

## Request format

| Name              | Size (bytes) | Internal type      | Description                                                                  |
| ----------------- | ------------ | ------------------ | ---------------------------------------------------------------------------- |
| `attestationType` | 2            | `AttestationType`  | Attestation type id for this request, see `AttestationType` enum.            |
| `sourceId`        | 4            | `SourceId`         | The ID of the underlying chain, see `SourceId` enum.                         |
| `upperBoundProof` | 32           | `ByteSequenceLike` | The hash of the confirmation block for an upper query window boundary block. |

## Verification rules

Given the upper boundary for the query range (`upperBoundProof`), the confirmed block on the upper query window boundary is determined and provided in the response, together with the block timestamp. In addition, the number of confirmations that were used to determine the confirmation block is provided. Also average block production time in the query window is calculated and returned. Here we take the lowest and the highest (confirmed) block number of the query window and their respective block timestamps. The timestamps are in seconds, but the average block production rate is given in milliseconds.

``` text
                                        (highestBlock.timestamp - lowestBlock.timestamp) * 1000
averageBlockProductionTimeMs = floor(  ---------------------------------------------------------- )
                                            highestBlock.number - lowestBlock.number
```

## Response format

| Name                           | Type         | Description                                                          |
| ------------------------------ | ------------ | -------------------------------------------------------------------- |
| `blockNumber`                  | `uint64`     | Number of the transaction block on the underlying chain.             |
| `blockTimestamp`               | `uint64`     | Timestamp of the transaction block on the underlying chain.          |
| `numberOfConfirmations`        | `uint8`      | Number of confirmations for the blockchain.                          |
| `averageBlockProductionTimeMs` | `uint64`     | Average block production time based on the data in the query window. |

Next: [00004 - Referenced Payment Nonexistence](./00004-referenced-payment-nonexistence.md)

[Back to Home](../README.md)
