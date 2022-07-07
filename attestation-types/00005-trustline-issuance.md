# 00005 - Trustline Issuance

- id: 5
- name: `TrustlineIssuance`
- [Definition file](https://github.com/flare-foundation/attestation-client/blob/main/lib/verification/attestation-types/t-00005-trustline-issuance.ts)

## Description

The purpose of this attestation type is to provide information about a [token issuer](https://xrpl.org/trust-lines-and-issuing.html) on the XRP Ledger.

A successful attestation is provided by extracting the following data from an account's history:

1. [Payment](https://xrpl.org/payment.html) Transaction Data

The payment transaction "issues" the token on the XRP Ledger to a receiver who has created a trust line with the issuer. The attestator will provide the [token amount](https://xrpl.org/currency-formats.html#token-amounts) fields:

- [`currency`](https://xrpl.org/currency-formats.html#currency-codes)
- `value`
- `issuer`

2. [SetRegularKey](https://xrpl.org/setregularkey.html) Transaction Data

Attestor must confirm that the issuer has set the account regular key to `rrrrrrrrrrrrrrrrrrrrBZbvji` (AKA `[ACCOUNT_ONE](https://xrpl.org/accounts.html#special-addresses)`)

3. [AccountSet](https://xrpl.org/accountset.html) Transaction Data

Attestor must confirm that the following config is applied to [account settings](https://xrpl.org/accountroot.html#accountroot-flags):

- Enabled rippling (lsfDefaultRipple)
- Disabled master key (lsfDisableMaster)

Attestation request should provide top and bottom block hash (or block index).

Account history should have 1 `Payment` transaction, 1 `SetRegularKey` transaction, and 1 `AccountSet` transaction only, otherwise is proven to be invalid. Account history can be queried via the [`account_tx`]((https://xrpl.org/account_tx.html)) method.

## Request format

Beside the standard fields (`attestationType`, `sourceId` and `upperBoundProof`) the request for `TrustlineIssuance` attestation type contains in addition fields `IssuerAccount`.

| Name              | Size (bytes) | Internal type      | Description                                                                  |
| ----------------- | ------------ | ------------------ | ---------------------------------------------------------------------------- |
| `attestationType` | 2            | `AttestationType`  | Attestation type id for this request, see [`AttestationType`](./enums.md#attestation-type) enum.            |
| `sourceId`        | 4            | `SourceId`         | The ID of the underlying chain, see [`SourceId`](./enums.md#source-id) enum.                         |
| `upperBoundProof` | 32           | `ByteSequenceLike` | The hash of the confirmation block for an upper query window boundary block. |
| `issuerAccount`   | 20           | `ByteSequenceLike` | Ripple account address as bytes                                           |

## Verification rules

- There must be an account associated with provided address
- Address must be funded later than system bottom block number and upperBoundProof
- Call XRPL [`account_info`](https://xrpl.org/account_info.html) and check:
 
  - account must have rippling enabled (flag `lsfDefaultRipple` is set)
  - account must have disabled master key (flag `lsfDisableMaster` is set)
  - account must have set the regular key to account_one (`rrrrrrrrrrrrrrrrrrrrBZbvji`)

- There can only be two `Payment` transactions associated with this address
  - 1. Account funding transaction done in past 2 days
  - 2. Token issuance transaction from which we extract:
    - CurrencyCode
    - Value
    - Issuer

## Response format

| Name                   | Type         | Description                                                 |
| ---------------------- | ------------ | ----------------------------------------------------------- |
| `tokenCurrencyCode`    | `bytes32`    | 3 letter code or 160-bit hexadecimal string known as [Currency code](https://xrpl.org/currency-formats.html#currency-codes). The first byte indicates whether it is a 3 letter encoded ascii string `0x00...` or 160 bit hex string `0x01...`  |
| `tokenValueNominator`  | `uint256`    | Nominator of the token value described as the fraction reduced by the highest exponent of 10           |
| `tokenValueDenominator`| `uint256`    | Denominator of the token value described as the fraction reduced by the highest exponent of 10         |
| `tokenIssuer`          | `bytes32`    | Ripple account address of token issuer as bytes (right padded address bytes (20 + 12)) |

Next: [Add new one :D](../README.md)

[Back to home](../README.md)
