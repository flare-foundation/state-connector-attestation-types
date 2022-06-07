# 00005 - Trustline Issuance

- id: 5
- name: `TrustlineIssuance`
- [Definition file](https://github.com/flare-foundation/attestation-client/blob/main/lib/verification/attestation-types/t-00005-trustline-issuance.ts)

## Description

The purpose of this attestation type is to provide information about a issued trustline, its issuer account and its issued token. Note that this type of attestation is only available on XRP ledger

A successful attestation is provided by extracting the following data about a transaction:

- Issuance token currency code
- Account address
- regular key (has to be rrrrrrrrrrrrrrrrrrrrBZbvji - ACCOUNT_ONE)
- Indicator that account enabled rippling (lsfDefaultRipple)
- Indicator that account disabled master key (lsfDisableMaster)
- Indicator that Account has only one token payment transactions 
  - Currency code associated with this issued transaction
  - Value issued
  - Issuer address
- top block hash (can be number)
- bottom block hash (can be number)


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
- address had to be founded later that system bottom block number and upperBoundProof
- Call XRPL [`account_info`](https://xrpl.org/account_info.html) and check
 
  - account must have rippling enabled (flag `lsfDefaultRipple` is set)
  - account must have disabled master key (flag `lsfDisableMaster` is set)
  - account must have set the regular key to account_one (`rrrrrrrrrrrrrrrrrrrrBZbvji`)

- there can only be two payment transaction associated with this address
  - founding account transaction done in past 2 days
  - one token transfer transaction from which we extract
    - CurrencyCode
    - Value
    - Issuer

## Response format

| Name                   | Type         | Description                                                 |
| ---------------------- | ------------ | ----------------------------------------------------------- |
| `tokenCurrencyCode`    | `bytes32`    | 3 letter code or 160-bit hexadecimal string known as [Currency code](https://xrpl.org/currency-formats.html#currency-codes). The first byte indicates whether it is a 3 letter encoded ascii string `0x00...` or 160 bit hex string `0x01...`  |
| `tokenValueNominator`  | `uint256`    | Nominator of the token value described as the fraction reduced by the highest exponent of 10           |
| `tokenValueDenominator`| `uint256`    | Denominator of the token value described as the fraction reduced by the highest exponent of 10         |
| `tokenIssuer`          | `bytes32`    | Ripple account address of token issuer as bytes  |

Next: [Add new one :D](../README.md)

[Back to home](../README.md)
