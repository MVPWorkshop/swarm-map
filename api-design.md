FORMAT: 1A

# Market Access Protocol


The Swarm Fund MAP aims to allow:

- Qualification providers
- Token issuers
- Investors
- Wallets
- Exchanges

to easily integrate and add the compliancy layer to any cryptocurrency based digital asset.

# API structure

## URL and REST structure

MAP API uses HTTPS only. All requests in HTTP will be completely ignored.

The API relies on REST HTTP verbs to interact with the objects in the system. 

---

Resources are created/registered using the POST action.

        `POST /token-issuers`
        
If the resource already exists, it is returned with a `409 Conflicts` instead of a `201 Created` response code.

---

Resource data is retrieved using the GET action.

        `GET /token-issuers/<pubKey>`
        
--- 

Relationships are created between resources using the PUT action. The exact resource on which the relationship will be attached is implicitly recognized from the successfuly authenticated user.

        `PUT /token-issuer/<pubKey>`


---

`DELETE /qualification-providers/<pub>/certs/<uid>`


## Response structure


Standard response structure will include the following header:

    `Content-Type: application/json`
    

## Data types

The response body will include a **JSON** encoded object of the resource requested.
The JSON structure returned is limited to primitives (boolean, number, string) and compounds (array, objects, null).

Date and DateTime elements are explicitly expected to be formatted in **ISO8601**.

DateTime elements accept a timezone in input but will always return a **UTC** formatted **ISO8601** string.

### Response codes and errors


**200**: OK - the action was executed and completed

**201**: Created - standard response to a POST action which also indicates that resource creation is also completed

Errors are indicated by http response code along with a corresponding human readable message. Additional details will be attached as a JSON encoded resource structure.

Standard errors are:

**400**: Parameter validation failed
            
**401**: Authentication headers required or failed
    
**403**: Requested resource is not available for the currently authenticated user

**404**: Resource not found

**405**: Method not allowed

**409**: Conflict

**429**: Too many requests, slow down and try again later

**422**: Unprocessable Entity due to payload being too large in size.

**500**: Resource action failed due to internal error

**503**: Resource is currently unavailable due to maintenance or overloading, please try again later

## API versioning

The MAP API currently does not support versioning.

Versioning will be supported once there is no way around making breaking changes to develop or improve a feature.

# Authentication

All users using the MAP API are authenticated using their public keys, details of authentication mechanism will be determined.

For now, we will assume use of provided headers.

Whenever authentication is needed, these HTTP headers are required:

| Header name      | Description                                                                                   | Example                            |
|------------------|-----------------------------------------------------------------------------------------------|------------------------------------|
| X-Swm-Blockchain | The underlying blockchain of the wallet, needed to determine the exact signing algorithm used | Bitcoin                            |
| X-Swm-Address    | The public key of the wallet                                                                  | 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN |
| X-Swm-Signature  | The wallet address signed by the wallet's private key                                         |  NG8tFK3dNwQaiYA0VOu...            |


> ***WARNING:*** We **strongly** recommend generating a new wallet when first registering with MAP to prevent the case in which the signed public key which is compromised from a third-party service which uses a similar authentication scheme.

***NOTE:*** Sucessfully authenticated `PUT` requests will attach a relationship to the resource owned by the authenticated user. 

# Getting started

## Qualification provider

The qualification provider is the user which is able to provide KYC-like compliance checks and thus is vital in the process of qualifying investors to manage a certain digital asset.

**Steps to register as a qualification provider:**

1. [Register](#reference/qualification-provider/account/register)
2. [Get Qualification Provider](#reference/qualification-provider/api-calls/get-qualification-provider)
2. [Certificates registration](#reference/qualification-provider/certificates/create-certificate)
3. [List of all certificates](#reference/qualification-provider/certificates/get-certificate-list)
4. [Get Certificate](#reference/qualification-provider/certificates/get-certificate-by-uid)
5. [Delete certificate](#reference/qualification-provider/certificates/delete-certificate)
5. [Register claim about investor](#reference/qualification-provider/claims/register-claim-about-investor)

<!--![Qualification Provider Flow](http://swarm-map-public-images.s3-website.us-east-2.amazonaws.com/Qualification-provider-light.svg)-->

## Token issuer

The token issuer is the user which is able to create an asset on the MAP blockchain.

1. [Register](#reference/token-issuer/api-calls/register)
2. [Get token issuer](#reference/token-issuer/api-calls/get-token-issuer)
3. [Create a new asset](#reference/token-issuer/api-calls/create-asset)
4. [List of wallet connection requests](#reference/token-issuer/api-calls/list-of-wallet-connection-requests)
5. [Get wallet connection request](#reference/token-issuer/api-calls/get-wallet-connection-request)
6. [Update wallet connection request](#reference/token-issuer/api-calls/update-wallet-connection-request)
7. [Update token DNA](#reference/token-issuer/api-calls/update-token-dna)

<!--![Token Issuer Flow](http://swarm-map-public-images.s3-website.us-east-2.amazonaws.com/Token-issuer-light.svg)-->

## Investor

The Investor is a person or an entity which wishes to buy tokens issued by the Token Issuer. 
Before the Investor can buy the tokens, or in other words *invest*, a Qualification provider must acknowledge that the investor is compliant to invest in the token, according to the rules specified by the Token issuer.

1. [Send wallet connection request](#reference/investor/send-wallet-connection-request/send-wallet-connection-request)
2. [GET Authorization signature](#reference/investor/get-authorization-signature/get-authorization-signature)

<!--![Investor Flow](http://swarm-map-public-images.s3-website.us-east-2.amazonaws.com/Investor-light.svg)-->


## API integration

For using the SDK and MAP API please consult the [MAP API Docs](/docs/api).

## MAP Blockchain

To keep track of Assets, Qualification Providers, Qualifications and Associations and at the same time having these information remain private and secure MAP will be built as a decentralized network of nodes having a blockchain as a fundamental layer. 
All actions triggered via API are in fact at the end blockchain transactions and are associated with each address (e.g. Token Issuer's).

The Swarm Fund MAP blockchain is an immutable store of records that has two basic types of transactions Qualification and Association.


## Transfer restriction rules


Transfer restriction rule section of Token DNA document describe rules every transfer have to abide to be allowed. Because rules can include properties that are only known by the token contract itself in the time of transfer, rules are defined in two groups: rules to be checked on-chain where Asset token is deployed, and rules that are to be checked on MAP node.


### Transfer authorization flow


Before any transfer on Asset, the token can be executed it needs to comply to transfer rules, listed in Token DNA. Token Asset contract should be set up to be aware of the existence of a contract or MAP transfer rules in Token DNA.

If the rules in token DNA are including ones checked on MAP node, transfer data sent to Asset Token has to include Authorization signature from MAP node, proving rules have been validated. Example contract’s transfer method should have at least following signature:

```
transfer(address _to, uint256 _value, uint256 _nonce, uint256 _expiration, bytes32 msgHash, uint8 v, bytes32 r, bytes32 s) public returns (bool success)
```

where msgHash consists of `(asset, from, to, data, nonce, expiration)` values and expiration hold time validity of authorization. 

If the rules in token DNA are including ones checked on Asset Token blockchain, implemented transfer method on Asset token contract will check them also before allowing transfer to be executed.

MAP authorization rules declaratively define rules for transfer to be allowed using properties of Asset, Transfer and Transfer Context (External properties?).

####Asset properties `(<token> prefix)`

Stable properties acquired from Token DNA document or Token contract.
1. Creation time
2. Supply
3. Decimals
4. Blockchain
5. Network

####Transfer properties `(<transfer> prefix)`

Properties have taken from transfer request.
1. Asset
2. From
3. To
4. Value


####Transfer context `(<ctx> prefix)`

Acquired by MAP node accepting authorization request.
1. Nonce
2. Expiration
3. IP
4. Timestamp
5. GEOIP

#### MAP properties `(<map> prefix)`

Properties MAP holds on about objects such as wallets, assets, token issuers, qualification providers, etc.

### Transfer restriction rules

As stated earlier, two sets of transfer restriction rules are defined in token DNA:

1. Rules enforced on token contract
2. Rules enforced by MAP

For now, the first set of rules are defined by a single address of contract enforcing the rules, and MAP enforced rules are listed in token DNA document. Example of rules section of Token DNA:

```
transferRules: {
    contract: {
        address: "0x..."
    },
    map: {
        type: "cert-list",
        version: 1,
        description: "https://swarm.fund/assets/SWT/rules.html"
        rules: [{
            ...
        }]
    }
}
```

### MAP transfer restriction rules

Map transfer restriction definition section of transfer rules has the following properties:

1. `type`: defines the type of rules listed in rules property, allowing a token issuer to choose how rules are going to be defined, using different implementations to check them. For example, “cert-list” type will just list all certification types wallets needs to have for simple rule engine to check, but “swarm-rules” can include much more complex rules.
2. `version`: maybe not needed as type can handle versioning issues.
3. `description`: URL of a document describing contained rules to investors. 
4. `rules`: JSON document with a definition of rules for the type.

Map node needs to have an implementation of rule checks for transfer_rules.map.type, or transfer restriction authorizations will return an error indicating unknown rule type in token DNA.

## Token DNA document

Token DNA document has, at least, following top level properties:

1. `token`: listing token properties which can be used on MAP node.
2. `properties`: various token properties, like an icon, description, etc.
3. `transferRules`: properties defining on-chain and MAP transfer rules.

Example of token DNA document follows.

```
{
    token: {
        blockchain: Ethereum,
        network: 1,
        address: "0x...",
        name: "Swarm token",
        symbol: "SWT",
        decimals: 18,
        supply: 10000000,
        created: "2019-02-19T12:04:50+00:00"  
    },
    properties: {
        ...
    },
    transferRules: {
    contract: {
        address: "0x..."
    },
    map: {
        type: "cert-list",
        version: 1,
        description: "https://swarm.fund/assets/SWT/rules.html"
        rules: [{
            ...
        }]
    }
}
```




# Group Qualification Provider

Qualification providers are entities which can verify information about identities and issue different types of trusted certificates. Qualification provider defines details about certificates and the required data subject need to provide to obtain one. One example of such certificates would be KYC certificates that can be issued to prospective investors. Qualification providers must register themselves and update a list of their certificates 
on MAP, with certificates identified by unique id.

Instances of certificates issued to subject identities are Claims. Information about the existence of Claim is
registered on MAP, along with basic data required to check the availability of registered Claim. The claim is identified by a certificate's unique id and subject's public key.

## API Calls [/qualification-providers]

### Register [POST]

Registres new Qualification Provider.

+ Request (application/json)

    + Headers
        
            X-Swm-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                name: "The user-provided arbitrary name of the qualification provider",
                publicKey: "{publicKey}",
            }
    
    + Attributes

        + name: `The user-provided arbitrary name of the qualification provider.` (required)
        + publicKey: `public key (ie. ED25519) QP will use to sign objects.` (required)

+ Response 201 (application/json)

    + Attributes (Qualification provider)            

## Get Qualification Provider [GET /qualification-providers/{pubKey}]

Retreive Qualification Provider object.

+ Request

    + Parameters
    
        + pubKey: Ed25519 public key QP will use to sign objects.

+ Response 200 (application/json)

    + Attributes (Qualification provider)
    
## Certificates [/certificates]

Certificate is a verification about some identity that can be issued by Qualification Provider. 

### Create Certificate [POST /qualification-providers/{pubKey}/certificates]

Register possible Certificate by Qualification Provider on MAP.

+ Request (application/json)
    
    + Headers

            X-Swm-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                description: "Description of certificate",
                constraints: {
                    ...
                }
            }
            
    + Attributes
            
        + description: Holds specific details of this qualification. (required)
        + constraints: Additional constrainst that may apply to the certificate. (optional)

+ Response 201

    + Attributes (Certificate)
    

### Get Certificate List [GET /qualification-providers/{pubKey}/certificates/]

List Certificates for requested Qualification Provider.

+ Request (application/json)

+ Response 200

    + Attributes (array[Certificate])
    

### Get Certificate by UID [GET /qualification-providers/{pubKey}/certificates/{uid}]

Get details about specific certificate.

+ Request (application/json)

+ Response 200

    + Attributes (Certificate)
    

### Delete Certificate [DELETE /qualification-providers/{pubKey}/certificates/{uid}]

Remove Certificate registration. Further Claim registration on this Certificate is not possible, but issued claims using this Certificate remains valid until specific invalidation.

+ Request (application/json)

+ Response 200

## Claims [/claims]

### Register claim about investor [POST /qualification-providers/{pubKey}/claims]

Register issuance of Claim for specific Investor and Certificate.

+ Request (application/json)
    + Headers

            X-Swm-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                issuer: "",
                expirationDate: "",
                investor: "",
                certificate: ""
            }
            
    + Attributes
            
        + issuer: Qualification provider public key. (required)
        + expirationDate: Expiration datetime. (required)
        + investor: Investor public key. (required)
        + certificate: Certificate UID. (required)

+ Response 201

    + Attributes
        
        + uid: Generated claim uid.
        + issuer: Qualification provider public key.
        + expirationDate: Expiration datetime.
        + investor: Investor public key.
        + certificate: Certificate UID.


# Group Token Issuer

Token issuers are entities who control asset token operation and on MAP, and are primarily responsible for asset and investor's wallet registration. They need to register themselves on the MAP for other parties to authenticate their actions. 

Apart from simple registration of themselves and their assets, the most important task Token issuers are handling
is responding to Investor's wallet registration requests - separating Investor's Wallet from Investor's claims. 
Token issuers are only entities in MAP in possession of Investor-Wallet connection information, handling registration tasks and saving connect information off-site.

## API Calls [/token-issuers]

### Register [POST]

Registers the token issuer.

+ Request (application/json)

    + Headers
        
            X-Swm-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Attributes
    
        + name: `Token issuer name` (string, required)
        + publicKey: `Public key` (string, optional)

+ Response 201 (application/json)

    + Attributes (Token issuer)
    
### Get Token Issuer [GET /token-issuers/{pubKey}]

Gets the information about Token Issuer.

+ Response 200 (application/json)

    + Attributes (Token issuer)
    
### Create Asset [POST /token-issuers/{pubKey}/assets]

Adds a new Asset to the MAP blockchain.

+ Request (application/json)

    + Headers
        
            X-Swm-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                address: ""
                name: "",
                description: "",
                symbol: "",
                decimals: 0,
                supply: 0,
                dna: {
                    hash: "",
                    url: ""
                }
            }
    
    + Attributes
    
        + address: `Token contract address` (string, required)
        + name: `The asset name` (string, required)
        + description: `The description of asset` (string, required)
        + symbol: `Token symbol` (string, required)
        + decimals: `Token decimals` (number, required)
        + supply: `Initial token supply` (number, required)
        + properties: `Some other fields that will be inserted` (string, optional)
        + dna: `Asset description document` (Dna, required)

+ Response 201 (application/json)

    + Attributes (Asset)
    
    
### List of wallet connection requests [GET /token-issuers/{pubKey}/assets/{pubKey}/requests?status={status}]

List wallet connection requests.

+ Request

    + Parameters
    
        + status: `type of status` (string, optional)

+ Response 200 (application/json)

    + Attributes (array[string])
    
    
### Get wallet connection request [GET /token-issuers/{pubKey}/assets/{pubKey}/requests/{uid}]

Retrieve specific wallet connection request.

+ Response 200 (application/json)

    + Attributes
        + request: `Encrypted investor request.` (string, required)

### Update wallet connection request [PUT /token-issuers/{pubKey}/assets/{pubKey}/requests/{uid}]

Update specific wallet connection request. This call is used to reassign Identity claims to Wallet. 

+ Request 
    + Body 
        {
            status: "",
            claims: []
        }
    
    + Attributes
        + status: `Status of request` (string, required)
        + claims: `Array of claims for wallet` (array[string], required)

+ Response 200 (application/json)

### Update token DNA [PUT /token-issuers/{pubKey}/assets/{pubKey}/dnas]

Update token DNA document properties. 

+ Request 
    + Body 
        {
            hash: "",
            url: ""
        }
    
    + Attributes (Dna)

+ Response 200 (application/json)


# Group Investor

Investors, identified by their MAP public keys, are entities expected to obtain certificates from Qualification
providers in form of MAP registered claims and participate in Assets trading by registering Wallets with Token issuers. 


Investor's Wallet Registration
Protection of Investor's privacy is accomplished by separating Wallet and Investor objects, and keeping that
connection information off-site MAP at Token Issuer records. Wallet registration flow is facilitating that
separation:

1. An investor sends Wallet registration request for selected Asset to MAP, encrypting the whole request object in such a manner that only destination Token Issuer can read information about Investor and connecting Wallet.

2. Token issuer handles the request by recording Investor-Wallet connection for possible future regulation 
queries and reissuing Investor claims with the same certificates and properties, connecting to new Wallet's 
public key.

Any entity wishing to check Wallet's certificates on particular Asset will use Token Issuer claims about 
Wallet, extending trust from Qualification Provider to Token Issuer of particular Asset.

### Send wallet connection request [POST /token-issuers/{pubKey}/assets/{pubKey}/requests]

Send register wallet request with Asset to token Issuer. Encrypted request data object consists of:

1. Investor public key
2. Details of wallet (blockchain, address)
3. Investor signature
4. Wallet signature
5. Decrypted Investor claim properties

Request data object is encrypted with designated Asset's Token Issuer key.

+ Request (application/json)

    + Body
    
            {
                request: "encrypted request data",
            }
    
    + Attributes

        + request: `Encrypted request data.` (string, optional)

+ Response 200 (application/json)
+ Response 404 (application/json)
+ Response 500 (application/json)


### Get Authorization signature [GET /authotizations]

Wallet owners, before sending the transaction to Token contract that has defined MAP transfer restrictions, have to acquire transfer authorization from MAP, when all rules will be checked for authorization eventually returned to wallet owner.

Authorization returned needs to carry relatively short expiration time, to prevent user pre-acquiring them.


+ Request (application/json)

    + Body
        {
            asset: "",
            from: "",
            to: "",
            value: ""
        }
        
    + Attributes
        + assets: `UID of asset` (string, required)
        + from: `Asset's wallet` (string, required)
        + to: `Asset's wallet` (string, required)
        + value: `Transfer value` (string, required)

+ Response 200
    + Attributes
        + assets: `UID of asset` (string, required)
        + from: `Asset's wallet` (string, required)
        + to: `Asset's wallet` (string, required)
        + value: `Transfer value` (string, required)
        + authorization: `signed authorization data <asset/from/to/value/nonce/expiration>` (string, required)
        + nonce: `Incremented nonce` (string, required)
        + expiration: `Rcpiration of authorization` (string, required)
        

      

# Group Exchange

Excanges use MAP to get Claims for specified Investors.

### Get Investor Claim [GET /claims/{pubKey}:{uid}]

Retreive Certificate Claim for Investor.

+ Request (application/json)
    + Attributes

        + pubKey: `Investor MAP public key` (string, required)
        + uid: `Certificate uid` (string, required)

+ Response 200 (application/json)
+ Response 404 (application/json)
+ Response 500 (application/json)
            

# Data structure


## Token issuer (object)

+ name: `The token issuer name` (string, required)
+ publicKey: `The public key of token issuer.` (string, optional)


## Asset (object)

+ uid: `The asset uid` (string, required)
+ name: `The user-provided arbitrary name of the asset` (string, required)
+ description: `The user-provided arbitrary description of the asset` (string, required)
+ dna: `The token DNA` (Dna, required)


## Qualification provider (object)

+ name: `The user-provided arbitrary name of the qualification provider.` (string, required)
+ publicKey: `Ed25519 public key QP will use to sign objects.` (string, required)


## Certificate (object)

+ uid: `Generated certificate uid` (string, required)
+ description: `Holds specific details of this qualification.` (string, required)
+ constraints: `Additional constrainst that may apply to the certificate.` (optional)

## Dna (object)

+ hash: `sha256 hash of document data.` (string, required)
+ url: `location of documents.` (string, required)


## Investor (object)

+ name: `The user-provided arbitrary name of the token issuer` (string, required)
+ ipfsMetadata: `The IPFS hash which contains arbitrary user-provided metadata.` (string, optional)
+ domain: `The fully qualified domain name which will be used for Qualification provider verification purposes.`
