# State Connector Attestation Types

Attestation providers play an essential role in the [attestation protocol](https://github.com/flare-foundation/attestation-client/blob/main/docs/attestation-protocol/attestation-protocol.md). They provide attestations for specific types of requests. These types are called **attestation types** and have to be designed in such a way that that they are **clear-cut decidable**. Clear-cut decidability includes the requirement of having a synchronized view on data from external data sources (e.g. other chains) that are used for data attestations. For example, in slower block producing blockchains like Bitcoin, one attestation provider may see a certain block at the moment of query while the other may not see it yet, as the block might not have been fully distributed throughout the network. Such providers could yield completely different attestations. Hence special data view synchronization protocols have to be considered.

Synchronized data views are also important due to representation of the submitted vote by the Merkle root of all attestations in the voting round. In case of non-synchronized data views, data providers would often vote differently on the most recent attestation requests (depending on the time of query), which would yield completely different Merkle roots and thus cripple the voting round, as achieving at least 50% of the same submitted Merkle roots would become extremely difficult. 

## Attestation type definitions

Each attestation type is defined by:
- **request format** - the data that is encoded into the attestation request bytes.
- **verification rules** - the rules attestation providers need to verify while querying specific external sources (e.g. blockchains), and 
- **response format** - defines which data (and data types) are obtained from the source queries and how the attestation hash is obtained, if attestation provider is able to verify the attestation request successfully. 

## Adding a new attestation type

In order to add a new attestation type, attestation provider community needs to approve it, implement the support for it in the [attestation client](https://github.com/flare-foundation/attestation-client) and decide from which voting round on it will be supported. To start the process, open an issue on this repository and propose a new type. Please also address the following points:
- Data sources: availability of external data sources, related costs and possible issues (data synchronization, availability, ...). In case of blockchains, relevant upgrades of [Multi-Chain-Client library](https://github.com/flare-foundation/multi-chain-client) should be considered.
- Users: describe the communities that will find the new attestation type useful and potential for growth.
- Technical definitions (request, response and verification rules - see the definitions of the existing types).
## Existing attestation types

The following attestation types are currently defined:
  - [00001 - Payment](attestation-types/00001-payment.md)
  - [00002 - Balance Decreasing Transaction](attestation-types/00002-balance-decreasing-transaction.md)
  - [00003 - Confirmed Block Height](attestation-types/00003-confirmed-block-height-exists.md)
  - [00004 - Referenced Payment Nonexistence](attestation-types/00004-referenced-payment-nonexistence.md)

## Additional resources

- [Attestation Client Suite](https://github.com/flare-foundation/attestation-client)
- [Multi-Chain-Client](https://github.com/flare-foundation/multi-chain-client)