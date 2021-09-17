- hip: XX
- title: Wallet Provider Schema
- author: [rocketmay](https://github.com/rocketmay), [0xJepsen](https://github.com/0xJepsen)
- type: Standards Track
- category: Application
- status: Draft 
- created: <07/03/21>
- discussions-to: <https://github.com/hashgraph/hedera-improvement-proposal/discussions/98>
- updated: <07/09/21> <07/08/21> <07/03/21>
- requires: 
- replaces: <HIP number(s)>
- superseded-by: <HIP number(s)>

## Abstract

This specification proposes a protocol called the Wallet Provider Schema (Abbreviated as WPS), for exchanging transactions between entities and web extensions. 

The WPS lays out the mechanisms for applications to find web extensions that implement it and establish connections to send transactions.

Entities which implement this protocol can be wallets, dApps, or other applications. The entity that sends the RPC call is referred to as the requesting entity, and the one that receives the RPC call and responds to it is referred to as the receiving entity.

This proposal does not require any new Hedera API endpoints. In addition, this protocol is designed to function without making any calls to the Hedera API. Any API calls are done outside the scope of this protocol.

This proposal will be implemented via a NodeJS javascript module. The module will be open source and installable as a public node module for ease of inclusion into applications. This will simplify implementation by developers and ensure that the standard can be updated as necessary.

## Motivation

This HIP describes a Wallet Provider Schema which describes a standardized method for web applications to identify web extensions installed on the user's computer which implement the WPS.

Developers are currently creating web authentication mechanisms from scratch for Hedera-based web apps. This limits consumer adoption and results in a poor user and developer experience. A standard protocol for decentralized applications to communicate with clients and allow clients to sign transactions would significantly improve the developer and user experience. The Hedera development space is still young, with many new projects developing promising ideas. These projects will need a standardized method of communication, much like what exists in the Ethereum space.

Taking the Metamask wallet as an example, Metamask offers a seamless way for users to interact with dApps, sending requests and signing transactions without compromising the security of their accounts. Another example is Wallet Connect, which is an open protocol for signing ETH transactions.

Without a communication standard, projects in the space are required to reinvent the wheel for every application and wallet. This adds overhead for the project team. This also represents a significant risk to the user who must reveal sensitive account information to web applications in order to interact with the Hedera network.


## Rationale

We propose establishing a standard protocol for web applications (the requesting entity) to identify web extensions (the receiving entity) installed on the user's computer that implement the WPS. Once the app is identified, message can be sent to the client application with transaction data to be processed (signed, returned and/or executed). There is no sensitive account information within the transaction data. 

The receiving entity recieves the transaction data and displays the request to the user. The user then approves or rejects the request, which the receiving entity processes. For example, in the case of a 'Sign' message, after approval the receiving entity signs it using their private key without exposing the key to the requesting entity. Per the specification of the 'Sign' message, the signed transaction is then returned to the requesting entity. 

This protocol is designed to allow any web extension to implement the WET Protocol and show up in a query by a web application, thus providing maximum flexibility for users and developers. It is similar to Wallet Connect, which is an open protocol. Contrast this with Metamask's protocol which is designed specifically for Metamask.


## Specification

JavaScript is the primary language that can communicate with the browser's [DOM](https://www.w3.org/TR/REC-DOM-Level-1/introduction.html#). It is thus necessary for this module to be a JavaScript module. The transaction transmission and signing process is governed by this protocol.

This is the generalized flow:

_Start of Protocol Scope_

Client Query Protocol
1. The requesting entity queries the browser for extensions which implement the WPS
2. The requesting entity generates a list of installed web extensions and the Account IDs that the extensions manage
3. The User selects the desired web extension/Account pair which they want to perform the transaction.
4. If the web extension does not already list the requesting entity as a connected application, the requesting entity sends a ConnectApplication RPC to the chosen extension.

Transaction Transfer Protocol
1. The requesting entity prepares the appropriate JSON-structured messages definied below.
2. The requesting entity sends the message to the selected web extension. 
3. The web extension receives the message and displays the information to the user, with the option to Approve or Reject the request.
4. The web extension processes the message appropriately and provides a return value to the requesting entity.

_End of Protocol Scope_


**Client Query Notes:**

The Client Query Protocol can be initiated at any time as deemed desirable by the requesting entity. For example it could be done upon loading a webpage, or when the user clicks on a 'Connect App' button, or right at the moment that a transaction needs to be signed and sent to the user. 

To be discussed prior to HIP formalization: We might consider implementing some method of encryption when sending the transaction information so that the transaction information is obscured. This is a privacy consideration rather than strictly a security one.

**Step 1**: 

Clients that implement the protocol will contain a 'HederaWPS' identifier in their manifest.json. The Requesting Website can then query for extensions which have the identifier and populate a list. 

An empty list signifies that there are no compatible browser extensions. It is then up to the Requesting Website if they want to provide options/instructions to the user. 

**Step 2**: The Requesting Website sends the message "requestAccounts" to the web extensions which implement HederaWPS.

Each Client implements RequestAccounts() which returns a set of Hedera Account IDs. They can either return the request automatically (since account ID's are typically not sensitive information), or the application can display a prompt to the user asking to release the accountIDs to the requesting application.

```
service RequestAccounts {
    rpc GetAccountList (AccountListQuery) returns (AccountListResponse) {
    }
}

message AccountListQuery{
    required bytes requestingApplication = 1; // unique identifier for the requesting entity
    optional bytes memo = 2;
}

message AccountListResponse{
    repeated AccountInfo accounts = 1;
    optional bool connected = 2; // True if the receiving entity was previously assigned to the requestingApplication via connectApplication RPC (Step 4)
    optional bytes memo = 3;
}

message AccountInfo {
    required bytes = 1;
    optional bytes memo = 2;
}
```

**Step 3**: The requesting entity should display the list of Accounts, sorted by Client. As part of this step the Requesting Website may query the network to obtain further information to assist the user in their choice (such as account balance), though this is not strictly part of the protocol.

The User will then select the Account which they want to perform the transaction with.

**Step 4**: The requesting entity sends the RPC "connectApplication" to the selected web extension.

Each Client implements ConnectApplication() which prompts the user to connect the web extension with the application - "(requestingApplication) wants to connect to (Web Extension Name). Approve / Reject" . Upon approval, the receiving application (web extension) saves the application identifier to its list of connected applications.

This serves two functions. First, it's a way for web extensions to conveniently 'remember' which sites they've been connected on and allow web applications to connect automatically when the user revisits the website.  It also serves as a way for web extensions to protect their users by (optionally) only accepting Transaction calls from sites they have been connected to. Further security features could be added in the future to verify the identity of requesting entities to decrease fraudulent transaction calls.
```
service ConnectApplication {
    rpc ConnectApplicationRequest (ConnectApplicationQuery) returns (ConnectApplicationResponse) {
    }
}

message ConnectApplicationQuery{
    required bytes requestingApplication = 1;
    optional bytes memo = 2;
}

message ConnectApplicationResponse{
    required bool approved = 1; // true if approved, false if rejected
}
```
**Transaction Transfer Notes:**

The Transaction Transfer Protocol occurs every time the web application wants to send a transaction to the receiving application. By necessity it can only occur once a web application has been selected from the Client Query Flow.

**Json Structures**

We prepose protobufs for four RPC calls and their corresponding requests and responses. The four calls are RequestLimit, Sign, Submit, and SubmitAndSign.

RequestLimit allows for information like MaxTransactionFee and PreAuthorizationLimit to be requested from entities.

This call can be used to query entities for their capabilities, to filter out entities which do not have the appropriate account permissions/settings or implement the required functions of the requesting entity. It can also potentially be used as a pre-authorization mechanism, for example a site that may prompt the user "This site wants to spend up to $50, is that OK?", and the wallet can then approve transactions up to that limit without interrupting the user.

Sign sends an unsigned transaction to an entity. The entity signs the transaction and returns the SigMap like that used in the hedera API to bytes of a transaction.

This call allows the requesting entity to generate a transaction, have it signed by the appropriate entity, then process and eventually execute the transaction on the HAPI. This gives the requesting entity control over the transaction and allows functionality such as multi-sig transactions to be fulfilled by the requesting entity.

Submit sends a transaction to an entity to submit to the HAPI. Any required signatures are included in the message.

This call allows for an entity to send a signed transaction to another entity for that entity to execute. The receiving entity returns the response from HAPI to the requesting entity.

SignAndSubmit sends a transaction to an entity, which is expected to sign the transaction and execute it on HAPI.

This is a streamlined call which reduces the number of messages sent between entities. The receiving entity returns the response from HAPI to the requesting entity.

message ResponseCode {
    uint64 id = 1
    oneof Response {
        Success = 0;
        Rejected = 1;
        UnrecognizedTransaction = 2;
        MaxTransactionFeeExceeded = 3;
        PreAuthroizationLimitExceeded = 4;
        CapabilityNotSupported = 5;
        TransactionException = 6;
        }
    Response response = 2;
}

message SignaturePair {
    bytes pubKeyPrefix = 1; // First few bytes of the public key
    oneof signature {
        bytes contract = 2; // smart contract virtual signature (always length zero)
        bytes ed25519 = 3; // ed25519 signature
        bytes RSA_3072 = 4; //RSA-3072 signature
        bytes ECDSA_384 = 5; //ECDSA p-384 signature
    }
}
message SignatureMap {
    repeated SignaturePair sigPair = 1; // Each signature pair corresponds to a unique Key required to sign the transaction.
}

message LimitsRequest{
    uint64 maxTransactionFee  = 1;
    uint64 preAuthorizationLimit = 2;
    bytes memo = 3;
}

message LimitResponse{
    ResponseCode response = 1;
    uint64 maxTransactionFee = 2;
    uint64 preAuthorizationLimit = 3;
    bytes memo = 5;
}

message SignRequest {
    bytes hash = 1; // correlation ID if not rpc
    bytes bodyBytes = 2; // hex
    bytes memo = 3;
}

message SignResponse {
    bytes hash = 1;
    ResponseCode response = 2;
    SignatureMap sigMap = 3; //matches HAPI sigMap , examples is above
    bytes memo = 4;
}

message SignAndSubmitRequest {
    bytes hash = 1;
    bytes bodyBytes = 2;
    SignatureMap sigMap = 3;
    bytes memo = 4;
}
message SignAndSubmitResponse { // for optimization over the wire --> less req res
    bytes hash = 1;
    ResponseCode response = 2;
    TransactionReceipt receipt = 3; // from HAPI
    bytes memo = 4;
}

message SubmitRequest {
    bytes hash = 1;
    bytes bodyBytes = 2;
    SignatureMap sigMap = 3;
    bytes memo = 4;
}

message SubmitResponse {
    bytes hash = 1;
    ResponseCode response = 2;
    TransactionReceipt receipt = 3; // from HAPI
    bytes memo = 4;
}

service Exchange {
    //unary
    rpc requestLimit(LimitsRequest) returns (limitResponse);
    rpc sign(signRequest) returns (signResponse); // returns transaction signed with ECDSA
    rpc submit(SubmitRequest) returns (SubmitResponse);
    rpc sign_and_submit(SignAndSubmitRequest) returns (SignAndSubmitResponse);
}

## Backwards Compatibility

This HIP is entirely opt-in and does not modify any existing functionality. It simply provides standards that client applications (such as wallets) and web applications can follow to interact with each other.

## Security Implications

Receiving entities are responsible for locally signing transactions. At no point are private keys ever shared or revealed to the requesting entity. This is the main purpose of this protocol. Because no sensitive account data is shared, account security through this protocol is maintained.

On the other hand, there are many considerations which developers should take into account when implementing this protocol into their applications:

Nothing can be done about a requesting entity (intentionally or not) generating an incorrect transaction. A malicious requesting entity can generate a transaction that is different than what the user is expecting. This protocol assumes that the receiving entity (web extension) properly unpackages the messages that it receives and displays the information in a readable, clear manner to the user for their review, and that the User is given accurate information and a clear indication of what action they are approving.

The permissions schema referenced below in the Open Issues section would provide more robust security for users.

## How to Teach This

Simple examples and guides will be incorporated into the existing Hedera documentation. Additionally, developer advocates can write educational content on the 'How to' of this feature. 

## Reference Implementation

To be developed.

## Rejected Ideas

N/A

## Open Issues

Open issues (not required for implementing the current HIP) allow users of web3 applications to provide partial information to the dApp—for example, implementing a wallet permissions schema and protocol. [EIP2255](https://eips.ethereum.org/EIPS/eip-2255) is a good resource for this.

## References

- [0] https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md
- [1] https://docs.hedera.com/guides/docs/hedera-api
- [3] https://eips.ethereum.org/EIPS/eip-1102#eth_requestaccounts
- [4] https://www.w3.org/TR/REC-DOM-Level-1/introduction.html#
- [5] https://github.com/NoahZinsmeister/web3-react
- [6] https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md
- [6] https://github.com/aragon/use-wallet
- [7] https://docs.metamask.io/guide/rpc-api.html#permissions

## Copyright/license

This document is licensed under the Apache License, Version 2.0 -- see [LICENSE](../LICENSE) or (https://www.apache.org/licenses/LICENSE-2.0)