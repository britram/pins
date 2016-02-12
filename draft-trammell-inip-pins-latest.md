---
title: Properties of an Ideal Naming Service
abbrev: PINS
docname: draft-trammell-inip-pins-latest
date: 
category: info

ipr: trust200902
area: Internet Architecture Board
workgroup: Names and Identifiers Program
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: B. Trammell
    name: Brian Trammell
    organization: ETH Zurich (D-INFK)
    street: Universitaetstrasse 6
    city: Zurich
    code: 8092
    country: Switzerland
    email: ietf@trammell.ch

normative:
  RFC2119:

informative:


--- abstract

This document specifies a set of necessary functions and desirable properties of an ideal system for resolving names to addresses and associated information for establishing communication associations in the Internet. For each property, it briefly explains the rationale behind it, and how the property is or could be met with the present Domain Name System. It is intended to start a discussion within the IAB's Names and Identifiers program about gaps between the present reality of DNS and the naming service the Internet needs by returning to first principles.

--- middle

# Introduction

[EDITOR'S NOTE: say nice things about DNS here]

# Functions

## Name to Address

Given a name, the naming service returns a set of addresses associated with that name, if such an association exists, where the association is determined by the authority for that name. Names may be associated with addresses in one or more address families (e.g. IP version 4, IP version 6). A querier may specify which address families it is interested in receiving addresses for, and the naming system treats all address families equally.

[EDITOR'S NOTE: DNS does this for the Internet via IN A and IN AAAA records.]

## Address to Name

Given an address, the naming service returns a set of names associated with that address, if such an association exists, where the association is determined by the authority for that address. 

[EDITOR'S NOTE: DNS does this for the Internet with IN PTR records within the in-addr.arpa. and ip6.arpa. zones. Note the limitation of delegation on octet (IPv4) and nibble (IPv6) boundaries. Cite workarounds.]

## Name to Name

Given a subject name, the naming service returns a set of object names associated with that name, if such an association exists, where the association is determined by the authority for the subject name.

[EDITOR'S NOTE: DNS does this via IN CNAME, but doesn't handle the set case, and there are restrictions on the use of IN CNAME (with respect to NS and MX records, but not SRV?)].

## Name to Auxiliary Information

Given a name, the naming service returns other auxiliary information associated with that name that is useful for establishing communication over the Internet with the entities associated with that name.

[EDITOR'S NOTE: Most other RRTYPEs implement this pattern.]

## Name/Address to Auxiliary Information

As a name might be associated with more than one address, auxiliary information as above may be associated with a name/address pair, as opposed to just with a name.

[EDITOR'S NOTE: DNS doesn't do this, does it?]

# Properties

The following properties are desirable in any service providing the functions in {{functions}}.

## Authority

Every association among names, addresses, and auxiliary data is subject to some authority: an entity which has the right to determine which associations between subjects and objects exist in its namespace. The following are properties of authorities in our ideal naming service:

### Federation of Authority

An authority can delegate some part of its namespace to some other subordinate authority. This property allows the naming service to scale to the size of the Internet, and leads to a tree-structured namespace, where each delegation is itself identified by a subject at a given level in the namespace.

### Unity of Authority

For a given subject, there is a single authority that has the right to determine the associations and/or delegations for that subject. The unitary authority for the root of the namespace tree may be special, though; see {{consensus-on-root-of-authority}}.

[EDITOR'S NOTE: The unitary authority for a given name in the DNS is its registry. The existence of registrars complicates this somewhat; see below.]

### Transparency of Authority

A querier can determine the identity of the authority for a given association.
An authority cannot delegate its rights or responsibilities with respect to a
subject without that delegation being exposed to the querier.

[EDITOR'S NOTE: It is very hard to enforce a restriction about delegations on the side (i.e. "I make this assertion 'cause somebody paid me to"). One could implement this in the current DNS by having the recursive also do a WHOIS, making information about the registrar available for local policy decisions.]

### Consensus on Root of Authority

Authority at the top level of the namespace tree is delegated according to a process such that there is universal agreement throughout the Internet as to the subordinates of those delegations.

[EDITOR'S NOTE: Today, this is the root zone. But note that this property does not necessarily imply, as with the present arrangement, a single authority at the root, only that the process by which the root is changed and operated leads to a universally consistent result.]

## Authenticity

A querier must be able to verify that the answers that it gets from the naming service are authentic.

### Authenticity of Federation

Given a delegation from a superordinate to a subordinate authority, a querier
must be able to verify that the superordinate authority authorized the
delegation.

[EDITOR'S NOTE: DNSSEC does this.]

### Authenticity of Response

The authenticity of every answer must be verifiable by the querier, and the
querier must be able to confirm that the answer is correct according to the
authority for the subject of the query.

[EDITOR'S NOTE: DNSSEC does this.]

### Authenticity of Negative Response

Some queries will yield no association, because no such association exists. In
this case, the querier must be able to confirm that the authority for the
subject of the query asserts that no association exists for the query.

[EDITOR'S NOTE: DNSSEC does this depending on how well you've set it up?]

## Consistency

[EDITOR'S NOTE: frontmatter here]

### Dynamic Consistency

When an authority makes changes to an association, every query for a given subject must give either the new valid result or a previously valid result, with known and predictable bounds on "how previously". Given that additions of, changes to, and deletions of associations may have different operational causes, different bounds may apply to different operations.

[EDITOR'S NOTE: TTL. Additions faster than changes and deletions, which is
probably the opposite of what you really want if you want name-service-based
revocation of things.]

### Explicit Inconsistency of Authoritative Results

Some techniques (e.g. DNS CDN) require giving different answers to different queries, even in the absence of changes: the stable state of the namespace is inconsistent. This inconsistency should be explicit: a querier should be able to know that an answer might be dependent on its identity, network location, or other factors.

[EDITOR'S NOTE: DNS doesn't do this. Not sure how useful this really is in a
world where people deploy anycast TCP.]

## Performance Properties

[EDITOR'S NOTE: note that these have to do with name service dynamics, and
that explicit tradeoffs here are possible and interesting.]

### Availability

The naming service as a while must be resilient to failures of individual
nodes providing the naming service, as well as to failures of links among
them. Intentional prevention of successful, authenticated query by an
adversary should be as hard as practical.

### Lookup latency

The time for the entire process of looking up a name and other necessary
associated data from the point of view of the querier, amortized over all
queries for all connections, should not significantly impact connection setup
or resumption latency.

### Bandwidth efficiency

The bandwidth cost for looking up a name and other necessary associated data,
from the point of view of the querier, amortized over all queries for all
connections, should significantly impact total bandwidth demand for an
application.

[EDITOR'S NOTE: What we mean here is that approaches that flood all name mapping updates to the entire Internet are probably not acceptable. Cite work on DNS traffic load to show that DNS has this property?]

### Query Linkability

It should be costly for an adversary to monitor the infrastructure in order to
link specific queries to specific queriers.

### Explicit Tradeoff

A querier should be able to indicate the desire for a benefit with respect to
one performance property by accepting a tradeoff in another, including:

- Reduced latency for reduced dynamic consistency
- Increased dynamic consistency for increased latency
- Reduced request linkability for increased latency and/or reduced dynamic consistency
- Reduced aggregate bandwidth use for increased latency and/or reduced dynamic consistency

# IANA Considerations

This document has no actions for IANA

# Security Considerations

[EDITOR'S NOTE: todo]

# Acknowledgments

Thanks to the Network Security Group at ETH Zurich, Andrew Sullivan, and
Suzanne Woolf for the input and discussions leading to this document.

