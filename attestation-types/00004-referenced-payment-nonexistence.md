# 00004 - Referenced Payment Nonexistence

- id: 4
- name: `ReferencedPaymentNonexistence`
- [Definition file](https://github.com/flare-foundation/attestation-client/blob/main/lib/verification/attestation-types/t-00004-referenced-payment-nonexistence.ts)

## Description

[Standardized payment references](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/payment-reference.md) are used to indicate special transactions. Usual use case involves a DeFi protocol that uses the attestation protocol, which requires from its users to make certain payments until certain deadline. Standardized payment reference is used for matching the payments to the paying users and their purposes. The purpose of this attestation type is to provide a proof that a transaction (payment) with a specific standardized payment reference, sending a specific amount of the native currency to a specific address was not carried out up to certain deadline. Such an attestation can be used as proof of a breach of payment obligations a user may have in relation to a DeFi protocol.

In some cases it could happen that a user tried to fulfil the obligation but failed due to causes outside their control (e.g. input transactions are blocked by the receiver). On some blockchains such transactions are recorded in blocks as failed transactions with special failure statuses, that indicate that the fault is on the receiving side and that the sender did all in their power to fulfil the obligation. See [here](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/transaction-status.md) for discussion on transaction status.

Attestations of this type are confirmed **if and only if** one of these two cases happens:

- The required transaction was not confirmed in due time.
- The required transaction was confirmed in due time, but it failed due to the sender's fault.

Note that if the sender tried to make the transaction in due time but the transaction failed due to receiver, the verification fails. In other words, the verification fails if the sender was successful in sending the required transaction (a transaction may fail and get recorded to the blockchain and the failure can clearly be attributed to the receiver).

Deadlines for a transaction to be attested can be provided in two ways:

- `deadlineBlockNumber`: The last block number to be considered as valid,
- `deadlineTimestamp`: The last block timestamp (in seconds since UNIX epoch) to be considered as valid.

Given a transaction in block `blockNumber` with `blockTimestamp` a transaction for which one of the following is true meets the criteria of being executed in time:

- `blockNumber <= deadlineBlockNumber`
- `blockTimestamp <= deadlineTimestamp`

An **overflow block** is any block for which the block number and the timestamp are both strictly greater than `deadlineBlockNumber` and `deadlineTimestamp`.

A [query window synchronization mechanism](https://github.com/flare-foundation/attestation-client/tree/main/docs/indexing/synchronized-query-window.md) uses the hash `upperBoundProof` of a confirmation block to synchronize the confirmed upper boundary block between attestation providers. For this attestation to be valid, an `upperBoundProof` must be provided in the request for a confirmed overflow block. Otherwise, the attestation is rejected.

Upon not being able to find the required transaction, a successful attestation is provided by extracting certain data from the indexer and request:

- Required deadline timestamp
- Required deadline block number
- Required destination address
- Required amount
- Required payment reference
- Lower query boundary block number
- Lower query boundary block timestamp
- First (lowest) overflow block number
- First (lowest) overflow block timestamp

In such a way the attestation confirms the required transaction did not appear in the interval between lower query boundary block (included) and the first overflow block (excluded).

## Request format

| Name                     | Size (bytes) | Internal type      | Description                                                                  |
| ------------------------ | ------------ | ------------------ | ---------------------------------------------------------------------------- |
| `attestationType`        | 2            | `AttestationType`  | Attestation type id for this request, see `AttestationType` enum.            |
| `sourceId`               | 4            | `SourceId`         | The ID of the underlying chain, see `SourceId` enum.                         |
| `upperBoundProof`        | 32           | `ByteSequenceLike` | The hash of the confirmation block for an upper query window boundary block. |
| `deadlineBlockNumber`    | 4            | `NumberLike`       | Maximum number of the block where the transaction is searched for.           |
| `deadlineTimestamp`      | 4            | `NumberLike`       | Maximum median timestamp of the block where the transaction is searched for. |
| `destinationAddressHash` | 32           | `ByteSequenceLike` | Hash of exact address to which the payment was done to.                      |
| `amount`                 | 16           | `NumberLike`       | The exact amount to search for.                                              |
| `paymentReference`       | 32           | `ByteSequenceLike` | The payment reference to search for.                                         |

## Verification rules

- The confirmed block that is confirmed by the confirmation block with the hash `upperBoundProof` must be an overflow block.
- Payment nonexistence is confirmed if there is no [native payment](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/native-payment.md) transactions with [standardized payment reference](https://github.com/flare-foundation/multi-chain-client/blob/main/docs/definitions/payment-reference.md) that meets all criteria for Payment attestation type (00001) and its transaction status is 0 (Success) or 2 (Failure due to receiver).
- If there exist only payment(s) with status 1 (failure, sender's fault) then payment nonexistence is still confirmed.

## Response format

| Name                          | Type      | Description                                                                                           |
| ----------------------------- | --------- | ----------------------------------------------------------------------------------------------------- |
| `deadlineBlockNumber`         | `uint64`  | Deadline block number specified in the attestation request.                                           |
| `deadlineTimestamp`           | `uint64`  | Deadline timestamp specified in the attestation request.                                              |
| `destinationAddressHash`      | `bytes32` | Hash of the destination address searched for.                                                         |
| `paymentReference`            | `bytes32` | The payment reference searched for.                                                                   |
| `amount`                      | `uint128` | The amount searched for.                                                                              |
| `lowerBoundaryBlockNumber`    | `uint64`  | The first confirmed block that gets checked. It is the lowest block in the synchronized query window. |
| `lowerBoundaryBlockTimestamp` | `uint64`  | Timestamp of the `lowerBoundaryBlockNumber`.                                                          |
| `firstOverflowBlockNumber`    | `uint64`  | The first (lowest) confirmed block with `timestamp > deadlineTimestamp` and `blockNumber  > deadlineBlockNumber`. |
| `firstOverflowBlockTimestamp` | `uint64`  | Timestamp of the `firstOverflowBlock`.                                                                |

Next: [00005 - Trustline Issuance](./00005-trustline-issuance.md)

[Back to home](../README.md)

