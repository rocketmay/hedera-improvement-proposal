---
hip: 10
title: Token Metadata JSON Schema
author: Sam Wood <sam.wood@luthersystems.com>, Susan Chan <susan.chan@luthersystems.com>, Stephanie Yi <stephanie.yi@luthersystems.com>, Khoa Luong <khoa.luong@luthersystems.com>, May Chan <may@hashpack.app>
type: Informational
needs-council-approval: No
status: Active
last-call-date-time: 2021-11-23T07:00:00Z
created: 2021-02-18
discussions-to: https://github.com/hashgraph/hedera-improvement-proposal/discussions/50
updated: 2021-05-12, 2021-12-06, 2022-02-19
---

## Abstract

This specification provides a standard scheme for non-fungible token metadata on HTS. The intent is to provide a flexible specification which will serve as the base format for all NFT's. 

NFT's minted to this specification will be able to be served by explorers, wallets and other applications, allowing those applications to display a minimum amount of information regardless of the use case of the NFT and providing a standardized format for specialized token types.

## Motivation

Token creators often desire to include an image and supplemental metadata that is associated with tokens, including Non-Fungible Tokens (NFTs). This specification provides a recommended standard for referencing and processing token metadata stored outside HTS.

This standard has been developed by the community with the intent of providing a flexible and robust schema for NFT metadata to address the following points:

1. Define a standard that is robust and flexible to allow the wide variety of NFT projects to be able to use it. 
2. Set a common 'base set' of fields that arbitrary token types can follow, which will allow wallets and galleries to be able to display a basic set of NFT data irregardless of the special features
3. Address the pitfalls of existing schemas which some projects have expressed problems with. For example, being able to serve data that isn't an image, or multi-file data.
4. Provide a mechanism for projects to specify their specific schema (the 'format' parameter), to allow for sub-schemas to be developed.

## Rationale

The token metadata has been developed with the input of over a dozen developers and projects in the NFT space. Many views and use cases were taken into consideration, such as:

1. Minimum required information to display any kind of NFT
2. Flexibility and Robustness
3. Compatibility with existing NFT standards (Ethereum and Solana's Open Sea standard)
4. Non-Image NFTs - Documents, Videos, Music, 3D Models
5. Multi-file NFTs

Rationale for specific fields is provided in section **Field Specific Rationale** below.

## Specification

Below is the human-readable schema, presented to maximize clarity. This document also includes a Formal JSON Schema definition to assist in data validation and describe the data in formal terms. See the **Formal JSON Schema Definition** section below.

```
{
    "name": "token Name - REQUIRED",
    "creator": "artist",
    "creatorDID": "DID URI",
    "description": "human readable description of the asset - RECOMMENDED",
    "image": "cid or path to the NFT, or if non-image, optional preview image - RECOMMENDED",
    "type": "mime type - ie image/jpeg - CONDITIONALLY OPTIONAL ",
    "files": [ // object array that contains uri, type and metadata
    {
        “uri”: “uri to file”,
		“type”: “mime type”,
        “metadata”: “uri to metadata”
    },
    … multiple …
    ],
    “format”: “format designation”
    "properties": {
    	// arbitrary json objects that cover the overarching properties of the token
    }
    "localization": { 
      // optional array of localization objects
    }
    // additional fields defined per the format
}
```
### Required, Optional and Conditionally Optional fields:

Name, description and image are the three basic fields in ERC721 NFT standards. Name is **required** for all NFT’s. Description and image are optional, but recommended to enable apps to display them better.

Type is listed as *Conditionally Optional*. If Image is defined, then type **must** be defined as well.

Creator, creator DID, attributes, files, properties and localization are optional and do not need to be included in the metadata.

### Field Specific Rationale

#### creator

A string for the creator, or for multiple creators to be attributed use comma separated values.

Eg. “John Doe” or “John Doe, Jill Doe”

#### creatorDID

This is an optional field to carry a decentralized identifier for the creator. This would allow a creator to subsequently lay claim to a creation even if the current ownership is different.

The DID resolves to a DID document with a public key, and the creator would prove ownership of the corresponding key with a signature.

For example, a Hedera DID looks like 'did:hedera:mainnet:7Prd74ry1Uct87nZqL3ny7aR7Cg46JamVbJgk8azVgUm;hedera:mainnet:fid=0.0.123'. The DID resolves to a JSON DID Document.

References: 
https://github.com/hashgraph/did-method/blob/master/did-method-specification.md
https://w3c.github.io/did-core/

#### image

Points to the uri of the image file. See **[uri formatting]** section for more details.

If one or more files are defined under “files”, “image” is considered to be the thumbnail that represents the collection of files. The term ‘thumbnail’ does not imply that the image size is reduced; “image” should always be the full resolution image. (Clarification added by Ashe Oro)

“image” was originally listed as ‘required’, however HashAxis noted that in the case of file NFTs, creating both image and file would require two files to pin instead of one. At the scale of millions, this doubling of the number of pinned images is significantly impactful to the cost of hosting the files. For files that can have their thumbnails generated programmatically, an image is not required. Because of this, the “image” field was changed to Conditionally Optional.

“image” is a standard field across other chains. There was discussion to change this to a more generic name such as ‘file’ or ‘uri’, however this would break cross-chain compatibility of this standard. Although image is defined as conditionally optional, ensure that it is defined in formats such as opensea where it is required.

#### type

Mime type of the image file. See **[mime formatting]** section for more details.

“type” is required if “image” is defined. 

Including the mime type allows applications to properly handle the file and greatly simplifies the code required to display the data.

#### files

“files” is an array of objects with the following format:
```
    {
        “uri”: “uri to file”,
        “type”: “mime type”,
        “metadata”: “metadata URI”
    }
```
“uri” is the uri to the file. See [URI Formatting]

“type” is required and is the mime-type of the file pointed to by the uri, see [Mime Formatting]

“metadata” is *optional*, and can contain additional information on the file. This is a URI that points to a metadata json file.

The opensea format defines “files” under the “properties” field. This standard places files in the root of the metadata in order to accommodate non-image NFT’s. It is recommended for NFT’s with the “opensea” format that they follow the opensea standard of placing files under the properties field, for cross-compatibility with other NFT’s, and not define a “files” field in the metadata root. The idea comes from the Solana NFT Standard from Metaplex: https://docs.metaplex.com/token-metadata/specification

#### format

For the optional fields of attributes and properties, as well as any additional fields above the required fields described in this schema, “format” defines the specific schema which is used by this NFT.
This allows NFT creators, communities and platforms to explicitly define the schema that they are using, which simplifies implementation for other projects hoping to use the same definition.

To reduce errors, “format” should be lower-case. For robustness, galleries and viewers should interpret format in a case-insensitive way to account for mistakes.

If format is not defined, it is assumed to be “opensea”, which is described here: https://docs.opensea.io/docs/metadata-standards . “opensea” can (and should) also be explicitly defined by format, where applicable, rather than leaving it blank. 

For the purposes of this base metadata schema we also define “format”: “none”, which is a catch-all that explicitly states that the NFT only follows the base fields of the schema but may define any number of additional fields.

#### properties

“properties” is defined as a collection of arbitrary fields and is the only optional field that is explicitly defined in the base schema.

The intention of “properties” is to provide a common place for information to be stored about the token. Future schema should use properties to include any additional information that is intended to be parsed by a generic text parser for display. For example, a “license” field could be defined with the value “Creative Commons Attribution 4.0 International”, and a gallery could parse through properties and display it dynamically without advance knowledge of the field.

It is not in the scope of this schema to define field naming standards or common fields. It is recommended for the community to create a standards body for this purpose.

Best Practice Recommendation: **It is strongly recommended that information such as ‘supply’, ‘royalties’ and other properties which are recorded on ledger should not be defined in the metadata.** This information is at best redundant and at worst can be factually incorrect.


#### Localization

Localization is an optional array of localization objects, which allow tokens/applications to present data uniformity across all languages.

Each localization attribute is a sub-object with two attributes: uri and locale.

uri - A URI to localized metadata in the specified locale. An application utilizing this metadata would discard the original metadata and use the localized metadata. It is the responsibility of the token creator to ensure that all non-localized fields and information is duplicated across the localized and base metadata.

locale - Specified locale, ie. en , fr , es

If localization is defined, an application may choose to use the localized metadata instead.

### Formatting Notes

#### URI Formatting

URI’s shall follow the following format: protocol://resource_location

For resources that are on the world wide web, the standard http and https protocols are acceptable. Ie. http://www.example.org/image/file.jpg

For resources that are on IPFS, the protocol must be ipfs:// and the resource location must be the cid of the file. Ie. ipfs://bafkreibwci24bt2xtqi23g35gfx63wj555u77lwl2t55ajbfjqomgefxce

The onus is placed on dApps to take the cid and access the file information through the method of their choosing.

CDN links such as Cloudflare and Infura are not acceptable. These are not primary sources. Ie. ~~https://cloudflare-ipfs.com/ipfs/bafkreibwci24bt2xtqi23g35gfx63wj555u77lwl2t55ajbfjqomgefxce~~

IPFS CIDS may contain file paths or extensions as long as they adhere to best practices as described here: https://docs.ipfs.io/how-to/best-practices-for-nft-data/#persistence-and-availability

For resources that are on the hedera file service, the protocol is hedera://

A more complete list of URI’s can be found here: https://en.wikipedia.org/wiki/List_of_URI_schemes

### Mime Formatting

Mime formatting shall follow the following format: type/subtype

As a rule, mime types are all lower case. However apps should be programmed to accept any case for robustness.

A list of common mime types can be found here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types

Note that mime types for directories are not uniformly defined. Some IPFS CIDs point to directories rather than files, so this type is useful. This standard shall define it using the format: text/directory 

## Reference Implementation

#### Example Schema: none

This is an example of a basic metadata as described by this schema. Format “none” is used to indicate that this metadata does not adhere to any specific sub-schema. Note that all the fields in properties are arbitrary.
```
{
    "name": "Example NFT",
    “creator”: “John Doe”
    "description": "This is an example NFT metadata",
    "image": "ipfs://bafkreibwci24bt2xtqi23g35gfx63wj555u77lwl2t55ajbfjqomgefxce",
    "type": “image/png”,
    "format": “none”,
    “properties”: {
        “license”: “MIT-0”,
        “collection”: “Generic Collection Name”,
        “website”: “www.johndoe.com”
    }
}
```
#### Example Schema: none, video

This is an example of a basic metadata as described by this schema. Format “none” is used to indicate that this metadata does not adhere to any specific sub-schema. Note that this NFT does not include an image, nor does it define any arbitrary properties.
```
{
    "name": "Example NFT",
    “creator”: “Jane Doe, John Doe”,
    "description": "This is an example NFT metadata",
    "format": “none”,
    “files”: [
        {
            “uri”: “ipfs://bawlkjaklfjoiaefklankfldanmfoieiajfl”,
            “type”: “video/mp4”,
            “metadata”: {
                “name”: “video name”,
                “description”: “nested file metadata”,
                “image”: “ipfs://bakcjlajeioajflakdjfneafoaeinovandklf”,
                “format”: “none”,
                “properties”: {
                    “additional_description”: “the image in this nested metadata is the video thumbnail.”
                }
            }
        }
    ]
}
```

#### Example Schema: none, video, multiple files, localization

An example similar to the above one. Format “none” is used to indicate that this metadata does not adhere to any specific sub-schema. The NFT does not contain a thumbnail, but some of the subfiles define their own thumbnails. The localization attributes would point to localized metadata.
```
{
    "name": "Example NFT",
    “creator”: “Jane Doe, John Doe”,
    “creatorDID”: “did:hedera:mainnet:7Prd74ry1Uct87nZqL3ny7aR7Cg46JamVbJgk8azVgUm;hedera:mainnet:fid=0.0.123”,
    "description": "This is an example NFT metadata",
    "format": “none”,
    “files”: [
        {
            “uri”: “ipfs://bawlkjaklfjoiaefklankfldanmfoieiajfl”,
            “type”: “video/mp4”,
            “metadata”: {
                "name": "Example Video"
            }
        },
        {
            “uri”: “ipfs://bawlkjaklfjoiaefklankfldanmfoieiajfl”,
            “type”: “application/pdf”,
            “metadata”: {
                "name": "Example second file",
                "description": "The description is recommended but optional. The image provided is an optional preview",
                "image": "ipfs://bawlkjaklfjoiaefklankflda1313ieiajfl",
                "type": "image/jpeg"
            }
        }
    ],
    "localization": [
        {
            "uri": “ipfs://bawlkjaklfjoiaefklankflda132zafga3tfa”,
            "locale": "es"
        },
        {
            "uri": “ipfs://bawlkjaklfjoiaefklankf12554wa6aga4fda”,
            "locale": "jp"
        }
    ]
}
```

#### Example Schema: opensea

This is an example of an opensea-compatible metadata that includes the important collectible NFT field “attributes” which is used for rarity. It also places the “files” array in the properties field, which is consistent with the opensea standard.
```
{
    "name": "Example opensea NFT",
    “creator”: “Jim Doe”,
    "description": "This is an example of an opensea NFT metadata",
    "image": "ipfs://bafkreibwci24bt2xtqi23g35gfx63wj555u77lwl2t55ajbfjqomgefxce",
    "type": “image/gif”,
    "format": “opensea”,
    "attributes": [
        {
            "trait_type": "coolness",
            "value": "50",
            "max-value": "100",
        },
        {
            "trait_type": "colour",
            "value": "red"
        },
    ],
    "properties": {
    "files": [
        {
            "uri": "https://www.image.net/abcd5678?ext=png",
            "type": "image/png"
        },
        {
            "uri": "https://www.arweave.net/efgh1234?ext=mp4",
            "type": "video/mp4",
        }
    ],
    "category": "video",
    "creators": [
        {
            "address": "xEtQ9Fpv62qdc1GYfpNReMasVTe9YW5bHJwfVKqo72u"
        }
    ]
    },
    "external_url": "https://www.exampleNFT.com/3",
}
```
## Formal JSON Schema Definition

The following is the formal definition of this schema using JSON Schema notation. JSON Schema assists in metadata validation and describes the data in the schema in more formal terms. **Despite looking like JSON, this is NOT how actual metadata should look. If you are creating the metadata file for an NFT do not use this definition as a reference.**

For more info see here: https://json-schema.org/
```
{
    "title": "Token Metadata",
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "Identifies the asset to which this token represents."
        },
        "description": {
            "type": "string",
            "description": "Describes the asset to which this token represents."
        },
        "image": {
            "type": "string",
            "description": "A URI pointing to a resource. Conditionally Optional."
        },
        "type": {
            "type": "string",
            "description": "Mime type of image. Required if image is defined."
        },
        "files": {
            "type": "object",
            "description": "Array of files.",
            “properties”: {
                “uri”: {
                    “type”: “string”,
                    “description”: “A URI pointing to a resource.”
                },
                “type”: {
                    “type”: “string”,
                    “description”: “Mime type of the resource.”
                }
                “metadata”: {
                    “type”: “string”,
                    “description”: “metadata for the file, following this standard”
                }
        },
        "format": {
            "type": "string",
            "description": "Name of the format used by the NFT."
        },
        "properties": {
            "type": "object",
            "description": "Optional. Arbitrary properties. Values may be strings, numbers, object or arrays."
        },
        "localization": {
            "type": "object",
            "description": "Array of localized metadata.",
            “properties”: {
                “uri”: {
                    “type”: “string”,
                    “description”: “A URI pointing to the localized metadata json file for this locale.”
                },
                “locale”: {
                    “type”: “string”,
                    “description”: “A two-letter locale.”
                }
        },            
    },
    “required”: [ “name” ]
}
```

## Backwards Compatibility

This HIP is entirely opt-in, and does not break any existing functionality. It simply provides standards to facilitate integratons for the display of metadata for HTS tokens.

## Security Implications

No known security concerns.

## How to Teach This

HTS implementations use this standard to provide additional metadata for their tokens. 

Wallet and token explorer implementations interrogate HTS tokens using this standard to display additional metadata for tokens.

Note that the referenced URI must fit within the token memo size restrictions.

## Rejected Ideas

This proposal originally only discussed NFTs and the JSON Metadata described in [0]. After discussion with the community [6] it became clear that this scope unnecessarily restrictive for HTS, and a JSON Metadata that supported fields beyond NFTs was required. The community further updated the HIP in February 2022 to address issues and further increase the flexibility and robustness of the specification.

## Open Issues

N/A

## References

[0] https://github.com/hashgraph/did-method/blob/master/did-method-specification.md
[1] https://w3c.github.io/did-core/
[2] https://docs.metaplex.com/token-metadata/specification
[3] https://docs.opensea.io/docs/metadata-standards
[4] https://docs.ipfs.io/how-to/best-practices-for-nft-data/#persistence-and-availability
[5] https://en.wikipedia.org/wiki/List_of_URI_schemes
[6] https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types
[7] https://json-schema.org/

## Copyright/license

This document is licensed under the Apache License, Version 2.0 -- see [LICENSE](../LICENSE) or (https://www.apache.org/licenses/LICENSE-2.0)
