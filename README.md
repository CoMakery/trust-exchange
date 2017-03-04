# ŦRUSŦ ΞXCHANGΞ

Latest build: [![Circle CI](https://circleci.com/gh/CoMakery/trust-exchange/tree/master.svg?style=svg)](https://circleci.com/gh/CoMakery/trust-exchange/tree/master)

Trust Exchange is:
  - An open protocol
  - A toolkit for building and reading distributed trust graphs
  - An ambitious plan to create interoperability between existing and future trust networks
  - Compatible with existing rating schemes (scores, percentages, star ratings, etc)
  - Open Source (MIT licensed)

![Trust Network Example](https://cdn.rawgit.com/CoMakery/trust-exchange/fee63549abcaa480ee18da207ebab7c45321de84/doc/images/network.png)

Trust Exchange is a very young codebase, so expect limited functionality, and don't use it in production just yet.

## Protocol: Trust Atoms

Trust Exchange is composed entirely of `Trust Atoms`, an intentionally open format which can naturally represent ratings and "vouches", as well as substantially more esoteric formats.

## Usage

### Create Claims

```
trust claim --help

Usage: trust-claim [options]

  Options:

    -h, --help                   output usage information
    --creator <creator>          DID or URL of claim creator
    --target <target>            DID or URL of claim target
    --description <description>  Rating description
    --tags <tag1, tag2>          Rating tags
    --value <value>              Rating weight in the range 0..1
    --algorithm <algorithm>      Signing algorithm
    --private-key <key>          Private key
```

For example:

```
trust claim \
  --creator did:00a65b11-593c-4a46-bf64-8b83f3ef698f \
  --target did:59f269a0-0847-4f00-8c4c-26d84e6714c4 \
  --description 'Elixir programming' \
  --value 0.99 \
  --algorithm EcdsaKoblitzSignature2016 \
  --private-key L4mEi7eEdTNNFQEWaa7JhUKAbtHdVvByGAqvpJKC53mfiqunjBjw
```

Creates the signed JSON-LD:

```json
{
    "@context": "https://schema.trust.exchange/TrustClaim.jsonld",
    "type": [
        "Verifiable Claim",
        "Rating"
    ],
    "issuer": "did:00a65b11-593c-4a46-bf64-8b83f3ef698f",
    "issued": "2017-03-02T22:07:32-08:00",
    "claim": {
        "@context": "https://schema.trust.exchange/GeneralRating.jsonld",
        "type": [
            "GeneralRating"
        ],
        "target": "did:59f269a0-0847-4f00-8c4c-26d84e6714c4",
        "rating": {
            "@context": "https://schema.org",
            "type": [
                "Rating"
            ],
            "author": "did:00a65b11-593c-4a46-bf64-8b83f3ef698f",
            "bestRating": 1,
            "worstRating": 0,
            "ratingValue": "0.99",
            "description": "Elixir programming"
        }
    },
    "signature": {
        "type": "sec:EcdsaKoblitzSignature2016",
        "http://purl.org/dc/terms/created": {
            "type": "http://www.w3.org/2001/XMLSchema#dateTime",
            "@value": "2017-03-03T06:07:32Z"
        },
        "http://purl.org/dc/terms/creator": {
            "id": "EcdsaKoblitz-public-key:020d79074ef137d4f338c2e6bef2a49c618109eccf1cd01ccc3286634789baef4b"
        },
        "sec:domain": "example.com",
        "signature:Value": "IJ9P6KQFWbYimr1tfWVg+Z6q9mPpz6KJ6jpd65U6BP5tRK1BMeCSXHuhSLOsS9zfwWd4pBOhxM7mEuVMVicq3+I="
    }
}
```

This structure reflects the [JSON-LD Verifiable Claim](https://opencreds.github.io/vc-data-model/#expressing-entity-credentials-in-json) structure.

We canonicalize the JSON, by minifying and sorting hashes by keys:

```json
{"@context":"https://schema.trust.exchange/TrustClaim.jsonld","claim":{"@context":"https://schema.trust.exchange/GeneralRating.jsonld","rating":{"@context":"https://schema.org","author":"did:00a65b11-593c-4a46-bf64-8b83f3ef698f","bestRating":1,"description":"Elixir programming","ratingValue":"0.99","type":["Rating"],"worstRating":0},"target":"did:59f269a0-0847-4f00-8c4c-26d84e6714c4","type":["GeneralRating"]},"issued":"2017-03-02T22:07:32-08:00","issuer":"did:00a65b11-593c-4a46-bf64-8b83f3ef698f","signature":{"http://purl.org/dc/terms/created":{"@value":"2017-03-03T06:07:32Z","type":"http://www.w3.org/2001/XMLSchema#dateTime"},"http://purl.org/dc/terms/creator":{"id":"EcdsaKoblitz-public-key:020d79074ef137d4f338c2e6bef2a49c618109eccf1cd01ccc3286634789baef4b"},"sec:domain":"example.com","signature:Value":"IJ9P6KQFWbYimr1tfWVg+Z6q9mPpz6KJ6jpd65U6BP5tRK1BMeCSXHuhSLOsS9zfwWd4pBOhxM7mEuVMVicq3+I=","type":"sec:EcdsaKoblitzSignature2016"},"type":["Verifiable Claim","Rating"]}
```

Then hash the canonical JSON to get an ID for the claim:

```
QmNu7NqYpPd2HSXiFVq2uBXwVvzzhEjbNBQktVfnLZb8KW  # sha2-256 multihash
```

### Retrieve Claims

***NOTE: claim retrieval/search is not yet implemented!***

Below is an API sketch; please submit any
comments or requests as Github issues.

```
trust get --help

Usage: trust-get [options]

  Options:

    -h, --help           output usage information
    --perspective <DID>  Perspective (identity) through which trust network is seen
    --creator <creator>  DID or URL of claim creator
    --target <target>    DID or URI of claim target
    --tags <tag1, tag2>  Filter by tags
    --depth <levels>     Crawls trust ratings to specified depth
    --min-value <value>  Min trust rating 0..1
    --max-value <value>  Max trust rating 0..1
```

```
trust map --help

Usage: trust-map [options]

  Options:

    -h, --help           output usage information
    --perspective <DID>  Perspective (identity) through which trust network is seen
    --creator <creator>  DID or URL of claim creator
    --target <target>    DID or URI of claim target
    --tags <tag1, tag2>  Filter by tags
    --depth <levels>     Crawls trust ratings to specified depth
    --min-value <value>  Min trust rating 0..1
    --max-value <value>  Max trust rating 0..1
    --summarize          Summarize claims / build analysis
    --falloff            Trust level relative to depth, eg: [1, 0.5, 0.33]
```

## Project History

The original vision for Trust Exchange was landed and anchored in 2006 by
[Harlan T Wood](https://github.com/harlantwood),
[Adam Apollo](http://www.adamapollo.com/),
and [Jack Senechal](https://github.com/jacksenechal),
and subsequently published on the
Enlightened Structure
site.

Harlan met [Noah Thorp](https://twitter.com/noahthorp) in 2007,
and their first conversation revolved around this kind
of decentralized trust technology.
In the summer of 2015, Noah, Harlan and [Joel Dietz](http://fractastical.com/)
created an open source, permissively licensed,
reference implementation of Trust Exchange, as a foundation for free, open,
interoperable trust systems.

In 2016, Noah and Harlan worked with
[Christopher Allen](http://www.lifewithalacrity.com/) and
[Manu Sporny](http://manu.sporny.org/) to bring
Bitcoin encryption to JSON-LD signed claims.  This work led to a
Portable Reputation Toolkit
[whitepaper](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2016/blob/master/final-documents/reputation-toolkit.pdf) and
[technical implementation](https://github.com/WebOfTrustInfo/portable-reputation-toolkit),
the latter of which was then folded back into a reboot of this codebase
in early 2017, notably centered around JSON-LD signed claims.
