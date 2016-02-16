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
    organization: ETH Zurich
    street: Universitaetstrasse 6
    city: Zurich
    code: 8092
    country: Switzerland
    email: ietf@trammell.ch

informative:
    RFC1035:

--- abstract

This document specifies a set of necessary functions and desirable properties of an ideal system for resolving names to addresses and associated information for establishing communication associations in the Internet. For each property, it briefly explains the rationale behind it, and how the property is or could be met with the present Domain Name System. It is intended to start a discussion within the IAB's Names and Identifiers program about gaps between the present reality of DNS and the naming service the Internet needs by returning to first principles.

--- middle

# Introduction

The Internet's Domain Name System (DNS) {{RFC1035}} is an excellent
illustration of the advantages of the decentralized architecture that have
made the Internet able to scale to its present size. However, the choices made
in the evolution of the DNS since its initial design are only one path through
the design space of Internet-scale naming services. Many other naming services
have been proposed, though none has been remotely as successful for general-
purpose use in the Internet.

This document returns to first principles, to determine the dimensions of the design space of desirable properties of an Internet-scale naming service. It is a work in progress, intended to start a discussion within the IAB's Names and Identifiers program about gaps between the present reality of DNS and the naming service the Internet needs.

{{query-interface}} and {{authority-interface}} define the set of operations a naming service should provide for queriers and authorities, {{properties}} defines a set of desirable properties of the provision of this service, and {{observations}} examines implications of these properties.

# Terminology

The following capitalized terms are defined and used in this document:

- Subject: A name, address, or name-address pair about which the naming service can answer queries

- Association: A mapping between a Subject and information about that Subject

- Authority: An entity that has the right to determine which Associations exist within its namespace

- Delegation: An Association that indicates that an Authority has given the right to make assertions about the Associations within the part of a namespace identified by a Subject to a subordinate Authority.

[EDITOR'S NOTE: need to make a terminology unification pass]

# Query Interface

At its core, a naming service must provide a few basic functions for queriers,
associating a Subject of a query with information about that subject. The
information available from a naming service is that which is necessary for a
querier to establish a connection with some other entity in the Internet,
given a name identifying it.

## Name to Address

Given a Subject name, the naming service returns a set of addresses associated
with that name, if such an association exists, where the association is
determined by the authority for that name. Names may be associated with
addresses in one or more address families (e.g. IP version 4, IP version 6). A
querier may specify which address families it is interested in receiving
addresses for, and the naming system treats all address families equally.

[EDITOR'S NOTE: DNS does this for the Internet via IN A and IN AAAA records.]

## Address to Name

Given an Subject address, the naming service returns a set of names associated
with that address, if such an association exists, where the association is
determined by the authority for that address.

[EDITOR'S NOTE: DNS does this for the Internet with IN PTR records within the in-addr.arpa. and ip6.arpa. zones. Note the limitation of delegation on octet (IPv4) and nibble (IPv6) boundaries. Cite workarounds.]

## Name to Name

Given a Subject name, the naming service returns a set of object names
associated with that name, if such an association exists, where the
association is determined by the authority for the subject name.

[EDITOR'S NOTE: DNS does this via IN CNAME, but doesn't handle the set case, and there are restrictions on the use of IN CNAME (with respect to NS and MX records, but not SRV?)].

## Name to Auxiliary Information

Given a Subject name, the naming service returns other auxiliary information
associated with that name that is useful for establishing communication over
the Internet with the entities associated with that name.

[EDITOR'S NOTE: Most other RRTYPEs implement this pattern.]

## Name/Address to Auxiliary Information

As a name might be associated with more than one address, auxiliary information as above may be associated with a name/address pair, as opposed to just with a name.

[EDITOR'S NOTE: DNS doesn't do this, does it?]

# Authority Interface

The query interface is not the only interface to the naming service: the
interface a naming service presents to an Authority allows updates to the set
of Associations and Delegations in that Authority's namespace. Updates consist
of additions of, changes to, and deletions of Associations and Delegations. In
the present DNS, this interface consists of the publication of a new zone file
with an incremented version number, but other authority interfaces are
possible.

# Properties

The following properties are desirable in a naming service providing the functions in {{query-interface}} and {{authority-interface}}.

## Authority

Every Association among names, addresses, and auxiliary data is subject to some Authority: an entity which has the right to determine which Associations and Subjects exist in its namespace. The following are properties of Authorities in our ideal naming service:

### Federation of Authority

An Authority can delegate some part of its namespace to some other subordinate
Authority. This property allows the naming service to scale to the size of the
Internet, and leads to a tree-structured namespace, where each Delegation is
itself identified with a Subject at a given level in the namespace.

### Unity of Authority

For a given Subject, there is a single Authority that has the right to
determine the Associations and/or Delegations for that subject. The unitary
authority for the root of the namespace tree may be special, though; see
{{consensus-on-root-of-authority}}.

[EDITOR'S NOTE: The unitary authority for a given name in the DNS is its registry. The existence of registrars complicates this somewhat; see below.]

### Transparency of Authority

A querier can determine the identity of the Authority for a given Association.
An Authority cannot delegate its rights or responsibilities with respect to a
subject without that Delegation being exposed to the querier.

[EDITOR'S NOTE: It is very hard to enforce a restriction about delegations on the side (i.e. "I make this assertion 'cause somebody paid me to"). One could implement this in the current DNS by having the recursive also do a WHOIS, making information about the registrar available for local policy decisions.]

### Consensus on Root of Authority

Authority at the top level of the namespace tree is delegated according to a
process such that there is universal agreement throughout the Internet as to
the subordinates of those Delegations.

[EDITOR'S NOTE: Today, this is the root zone. But note that this property does
not necessarily imply a single authority at the root as with the present
arrangement, only that the process by which the root is changed and operated
leads to a universally consistent result.]

## Authenticity

A querier must be able to verify that the answers that it gets from the naming
service are authentic.

### Authenticity of Delegation

Given a Delegation from a superordinate to a subordinate Authority, a querier
must be able to verify that the superordinate Authority authorized the
Delegation.

[EDITOR'S NOTE: DNSSEC does this.]

### Authenticity of Response

The authenticity of every answer must be verifiable by the querier, and the
querier must be able to confirm that the Association returned in the answer is
correct according to the Authority for the Subject of the query.

[EDITOR'S NOTE: DNSSEC does this.]

### Authenticity of Negative Response

Some queries will yield no answer, because no such Association exists. In
this case, the querier must be able to confirm that the Authority for the
Subject of the query asserts this lack of Association.

[EDITOR'S NOTE: DNSSEC does this depending on how well you've set it up?]

## Consistency

[EDITOR'S NOTE: frontmatter here]

### Dynamic Consistency

When an Authority makes changes to an Association, every query for a given
Subject must give either the new valid result or a previously valid result,
with known and predictable bounds on "how previously". Given that additions
of, changes to, and deletions of associations may have different operational
causes, different bounds may apply to different operations.

[EDITOR'S NOTE: DNS: TTL. Additions faster than changes and deletions, which is
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

[EDITOR'S NOTE: DNS aims to provide this through explicit secondaries and XFER, as well as through operational practice: e.g. through ease of anycasting UDP services.]

### Lookup Latency

The time for the entire process of looking up a name and other necessary
associated data from the point of view of the querier, amortized over all
queries for all connections, should not significantly impact connection setup
or resumption latency.

[EDITOR'S NOTE: DNS aims to provide this through being small and simple, and through the use of caching.]

### Bandwidth Efficiency

The bandwidth cost for looking up a name and other associated data necessary
for establishing communication with a given Subject, from the point of view of
the querier, amortized over all queries for all connections, should
significantly impact total bandwidth demand for an application.

[EDITOR'S NOTE: What we mean here is that approaches that flood all name mapping updates to the entire Internet are probably not acceptable. Cite work on DNS traffic load to show that DNS has this property?]

### Query Linkability

It should be costly for an adversary to monitor the infrastructure in order to
link specific queries to specific queriers.

[EDITOR'S NOTE: DPRIVE is working toward this.]

### Explicit Tradeoff

A querier should be able to indicate the desire for a benefit with respect to
one performance property by accepting a tradeoff in another, including:

- Reduced latency for reduced dynamic consistency
- Increased dynamic consistency for increased latency
- Reduced request linkability for increased latency and/or reduced dynamic consistency
- Reduced aggregate bandwidth use for increased latency and/or reduced dynamic consistency

[EDITOR'S NOTE: DNS doesn't do this, and can't really: TTL gives you a tradeoff knob but it's in the hands of the authority]

# Observations

We have shown that most of the properties of our ideal name service are met,
or could be met, by the present DNS protocol or extensions thereto. [EDITOR'S
NOTE: not yet, not really.] We note that there are further possibilities for
the future evolution of naming services meeting these properties.

[EDITOR'S NOTE: there are probably more than just this one, but this is the important one.]

## Delegation and Redirection are Separate Operations

Any system which can provide the authenticity properties in {{authenticity}}
is freed from one of the design characteristics of the present domain name
system: the requirement to bind a zone of authority to a specific set of
authoritative servers. Since the authenticity of delegation must be a
protected by a chain of signatures back to the root of authority, the location
within the infrastructure where an authoritative mapping "lives" is no longer
bound to a specific name server. While the present design of DNS does have its
own scalability advantages, this implication allows a much larger design space
to be explored for future name service work, as a Delegation need not always
be implemented via redirection to another name server.

# IANA Considerations

This document has no actions for IANA

# Security Considerations

[EDITOR'S NOTE: todo]

# Acknowledgments

This document is an output of a design work on naming services at the Network
Security Group at ETH Zurich. Thanks to the group, including Daniele Asoni and
Stephen Shirley, for discussions leading to this document. Thanks as well to
Andrew Sullivan and Suzanne Woolf for input and feedback.

