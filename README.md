# Related Web Properties

This specification defines the concept of Related Web Properties (RWPs), a way for web property owners to express a one-to-many directed relationship from one piece of web property to another piece of web property

## Definitions
*Web Property* - A hostname, IP address (or CIDR range), URI, IPFS CID, or IPNS name  
*Primary Web Property* - A Web Property which claims to be related to a set of Secondary Web Property  
*Secondary Web Property* - A Web Property which has been claimed to be related to by a Primary Web Property  

## Adopting RWPs

Currently, only a hostname may serve as the Primary Web Property, and may claim a relationship with Secondary Web Properties using one of two methods: the DNS method, or the Well-Known URI method.

### DNS method
When using the DNS method, one or more TXT records should be created with the following format on the Primary Web Property:

```
related-web-property=${property-type}=${serialized-property-value}
```

For example, Uniswap may set the following DNS records on `app.uniswap.org`:

```
related-web-property=hostname=uniswap.org
related-web-property=hostname=*.uniswap.org
related-web-property=ipns=app.uniswap.org
related-web-property=ipfs=QmNy6ppw64jmLBEZ6r8D19beUVH3objJPrjMfNxvugqakD
```

### Well-Known URI method
When using the Well-Known URI method, only the HTTPS scheme is acceptable. A file should be hosted at `/.well-known/related-web-properties.txt`, which conforms to the following rules:
1. Every line must either:  
    a. Be empty  
    b. Begin with a `#` character, which indicates a comment and should be skipped  
    c. Contain a valid RWP entry of the format `${property-type}=${serialized-property-value}`  
2. Lines must only end with LF, and not with CRLF  

For example, Uniswap may host the following resource at `https://app.uniswap.org/.well-known/related-web-properties.txt`

```
hostname=uniswap.org
hostname=*.uniswap.org
ipns=app.uniswap.org
ipfs=QmNy6ppw64jmLBEZ6r8D19beUVH3objJPrjMfNxvugqakD
```

## FAQs
### Why not require two-way attestation?
This specification is only concerned with allowing a single web property to make a set of claims about what is related to it. Determining which web properties is trustworthy is out of scope. Users who wish to consume this data have the option to simply trust the Primary Web Property to be secure and legitimate, or to require that both web properties claim each other as related.

### Why include the property type in the RWP entry?
This is to ensure that should future web property types be introduced which are not natively distinguishable, there still exists a mechanism to distinguish between them.

### Why are wildcards supported for the hostname web property type?
Similar to the Public Suffix List, there may be times when web property owners want to signal that only the root domain should be considered related, while subdomains may host User Generated Content (UGC). In this case, the non-wildcard variant should be used. There may be other times when the owner wants to indicate that both the root domain, but also all subdomains, should be considered related. In this case, both the non-wildcard and the wildcard variant should be used.

## Serializing RWPs

Each property type has a canonical serialization, given below.

### Hostnames
A hostname has property type `hostname`, and is serialized as follows:
1. If the hostname contains non-ASCII characters, apply the Punycode encoding algorithm  
2. Optionally, to indicate coverage of all subdomains, prepend a wildcard component to the hostname. Note that this is the only acceptable usage of the asterisk symbol

Examples:
- `hostname=example.com`  
- `hostname=*.example.org`  

### IP Addresses
An IP address has property type `ip` and is serialized as follows:
1. Serialize the IP address normally. If the IP address is IPv6, do not surround the IP address in square brackets  
2. Optionally, to indicate coverage of a range of IP addresses, use CIDR notation  

Examples:
- `ip=1.1.1.1`  
- `ip=127.0.0.0/8`  
- `ip=f9e9:574a:abba:8128:9ad5:4af6:aae8:da96`  

### URI
An URI as property type `uri` and is serialized as follows:
1. Remove the userinfo, query, and fragment components from the URI
2. Serialize the URI normally

Examples:
- `uri=https://example.shared.com`  
- `uri=custom://example.com`  

### IPFS
An IPFS CID has property type `ipfs` and is serialized directly.

Examples:
- `ipfs=QmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n`  
- `ipfs=bafybeihdwdcefgh4dqkjv67uzcmw7ojee6xedzdetojuzjevtenxquvyku`  

### IPNS
An IPNS name has property `ipns` and is serialized directly. Note that DNSLink names are supported

Examples:
- `ipns=k51qzi5uqu5dlvj2baxnqndepeb86cbk3ng7n3i46uzyxzyqj2xjonzllnv0v8`  
- `ipns=example.com`  
