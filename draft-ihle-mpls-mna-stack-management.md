---
title: "MPLS Network Action for Stack Management"
abbrev: "MNAMGMT"
category: std

docname: draft-ihle-mpls-mna-stack-management-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Multiprotocol Label Switching"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Multiprotocol Label Switching"
  type: "Working Group"
  mail: "mpls@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mpls/"
  github: "uni-tue-kn/draft-ihle-mpls-mna-lse-operations"
  latest: "https://uni-tue-kn.github.io/draft-ihle-mpls-mna-lse-operations/draft-ihle-mpls-mna-lse-operations.html"

author:
 -
    fullname: Fabian Ihle
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: fabian.ihle@uni-tuebingen.de
 -
    fullname: Michael Menth
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: michael.menth@uni-tuebingen.de

normative:

informative:


--- abstract
The MPLS Network Action (MNA) framework provides a general mechanism for the encoding and processing of network actions and their data.

This document introduces a network action to the MPLS Network Action (MNA) framework for stack management operations including MOVE-N-LSE and POP-N-LSE.
The MOVE-N-LSE operation elevates a specified number of LSEs within the stack facilitating a more efficient structure of the MPLS stack.
The POP-N-LSE operation removes a specified number of LSEs facilitating mechanisms such as Stateless MNA-based Egress Protection (SMEP).
--- middle

# Introduction

The MPLS Network Action (MNA) framework encodes network actions and their parameters in the MPLS stack.
{{?I-D.ietf-mpls-mna-hdr}} defines the encoding of such network actions and their data in LSEs.
These network actions are processed by all nodes on a path (hop-by-hop), by only selected nodes, or on an ingress-to-egress basis.

This document introduces a network action for stack management operations.
Those operations include a MOVE-N-LSE and a POP-N-LSE operation.
With the MOVE-N-LSE operation, a number of N LSEs from below the NAS are brought to the top-of-stack.
This can be leveraged to enable for more efficient MPLS processing with MNA.
An example for this is given in this document using a mechanism called HBH preservation.
With the POP-N LSE operation, a number of N LSEs below the NAS is popped.
This can be used to facilitate mechanisms such as Stateless MNA-based Egress Protection (SMEP).
SMEP provides an alternative to the rerouting mechanism defined for the PLR in {{?RFC8679}} allowing the PLR to be stateless.
In this context, the POP-N LSE operation eliminates the need to encode Bypass MPLS Labels (BML) in the ancillary data of the SMEP network action.
However, the stack management operations are not tied to specific use cases and may be used in a more generalized way.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Abbreviations
This document makes use of the terms defined in {{?I-D.ietf-mpls-mna-hdr}}.

Further abbreviations used in this document:

| Abbreviation |  Meaning                      |  Reference
| ---------- |  -------------------------------- |  -------------------
|    BML    |  Bypass MPLS Label |  {{?I-D.ihle-mpls-mna-stateless-egress-protection-00}}
|    SMEP    |  Stateless MNA-based Egress Protection |  {{?I-D.ihle-mpls-mna-stateless-egress-protection-00}}
{: #table_abbrev title="Abbreviations."}

# Stack Management Operations
This section describes the stack management network action encoding and the processing in an LSR.

## Network Action Encoding
The network action for stack management operations is encoded as follows:

- Network Action Indication: The stack management network action is indicated by opcode TBA1.

- Format: The stack management network action MUST be encoded using a Format B or a Format C LSE as defined in {{?I-D.ietf-mpls-mna-hdr}}, see {{fig-lseops-encoding-b}} and {{fig-lseops-encoding-c}}.

- Scope: The stack management network action can be used in any scope, i.e., in the select, HBH, and I2E scope. The scope depends on the use case.

- Ancillary Data: The stack management network actions requires four bits of in-stack ancillary data per operation (pop and move) to encode the parameter N.
The parameter N corresponds to the number of LSEs to apply the operation to.
For example, the POP-N-LSE operation with N=2 pops 2 LSEs that follow after the NAS.
In Format B, the four least-signifact bits of the ancillary data field contain the number of LSEs for the MOVE-N-LSE operation.
The next four bits contain the number of LSEs for the POP-N-LSE operation.
All other ancillary data bits MUST be set to zero and are reserved for future use.
For Format C, the same encoding of POP-N-LSEs and MOVE-N-LSEs applies to the first data field of the LSE.
The second data field, i.e., the 4-bit wide ancillary data field, MUST be set to zero and is reserved for future use.

- Post-Stack Data: None.

~~~~
{::include ./drawings/lseops-encoding-b.txt}
~~~~
{: #fig-lseops-encoding-b title="MNA for Stack Management Operations in Format B."}

~~~~
{::include ./drawings/lseops-encoding-c.txt}
~~~~
{: #fig-lseops-encoding-c title="MNA for Stack Management Operations in Format C."}

## Processing

A node that processes the stack management network action MUST perform the MOVE-N-LSE and POP-N-LSE operation according to the corresponding parameter N > 0.
If N = 0, the corresponding operation is not performed.

The ingress LER MUST ensure that no invalid state, e.g., a MOVE-N-LSE operation with N > \#LSEs below the NAS, results from the processing of the stack management network action.

# Use Cases and Examples
This section illustrates one use case for each operation in an example.

## POP-N-LSE: Stateless MNA-based Egress Protection (SMEP)
Stateless MNA-based Egress Protection (SMEP) is a proposed mechanism within the MNA framework {{?I-D.ihle-mpls-mna-stateless-egress-protection-00}}.
SMEP encodes egress protection information such as Bypass MPLS Labels (BMLs) in the MPLS label stack as network actions.
This approach eliminates the need for the Point of Local Repair (PLR) to maintain bypass forwarding states or engage in additional signaling for bypass tunnel mappings.
Instead, all necessary information is delivered to the PLR via the MPLS stack enabling failover to alternative egress paths in case of egress node or link failures.

However, in the proposed SMEP network action encoding, the 20 bit BMLs are split across the AD fields of a network action.
This requires the PLR to extract the BML from the network action first.
With the POP-N-LSE operation, the BMLs are contained as forwarding LSEs in the MPLS stack without the need to encode the bypass tunnel information in the AD fields of a network action.
An example is given in {{fig-smep}}.

~~~~
{::include ./drawings/smep.txt}
~~~~
{: #fig-smep title="Example using the POP-N-LSE operation for SMEP."}

The example network contains an LSP R1-R2-R3 and a bypass tunnel R2-R3'-R3''.
The label stack contains the forwarding labels L1, L2, L3, L3', and L3''.
If there is no egress failure, the LSR R2 executes the POP-N-LSE action and pops the BMLs L3' and L3''.
The packet is forwarded as usual according to the top-of-stack label L3.
If the LSR R2 detects an egress failure it becomes the PLR.
The POP-N-LSE action is ignored and the NAS is popped along with the top-of-stack label.
This time, the BMLs L3' and L3'' are at the top-of-stack.
The packet is forwarded according to those labels to the alternative egress node R3''.
This mechanism requires configuration or an indication to only execute the POP-N-LSE operation if there is no egress failure detected.

## MOVE-N-LSE: HBH Preservation
An efficient way to structure the MPLS stack with different NAS is to place to HBH-scoped NAS always below the top-of-stack forwarding label.
This reduces the required allocation of resources to a minimum as no irrelevant LSEs are parsed to find the HBH NAS located deeper in the stack.
Placing the HBH-scoped NAS close to the top and popping the top-of-stack forwarding label exposes the HBH-scoped NAS to the top which results in popping this NAS.
To prevent this, an LSR MAY bring the forwarding label from below the NAS to the top-of-stack using the MOVE-N-LSE operation.

In the following example, the stack management network action with the MOVE-N-LSE operation is added to the HBH-scoped NAS if one is present in the stack.
This brings one LSE from below the NAS to the top.
The MOVE-N-LSEs operation is only applied if the top-of-stack forwarding label is popped.
However, this mechanism poses a challenge when there are MNA-incapable nodes on the path.
An MNA-incapable node pops the top-of-stack forwarding label and exposes the HBH-scoped NAS to the top, leading to packet drop at the next hop.
To fix this, the MOVE-N-LSE operation can be leveraged to bring N LSEs at the preceding node of an MNA-incapable node to the top using a select-scoped NAS.
This way, forwarding labels for the upcoming N MNA-incapable nodes are brought to the top and the HBH-scoped NAS is never exposed to the top.
The MNA-capabilities are signaled to the ingress LER.
Therefore, the ingress LER places a select-scoped network action for the preceding node of MNA-incapable nodes in the MPLS stack.
No information about MNA capabilities of neighboring nodes is required in a transit node as all node capabilities are signalled to the ingress LER.

If multiple NAS with the stack management network action and N > 0 for the MOVE-N-LSE operation are present in the stack, all values for N are summed up and that number of LSEs below the last processed NAS is brought to the top.
An example is illustrated in {{fig-hbh-preservation}}.

~~~~
{::include ./drawings/hbh-preservation.txt}
~~~~
{: #fig-hbh-preservation title="Example using the MOVE-N-LSE operation."}

In {{fig-hbh-preservation}}, a MNA-capable node is followed by two MNA-incapable nodes.
Therefore, the stack management network action with N=2 for the MOVE-N-LSE operation is pushed by the ingress LER as a select-scoped NAS for the LSR R1.
Further, to enable the HBH preservation mechanism, the stack management action with N=1 for the MOVE-N-LSEs operation is added to the HBH-scoped NAS.
LSR R1 brings three LSEs to the top-of-stack, two from the MOVE-N-LSE operation in the select-scoped NAS and one from the HBH-scoped NAS.
At the MNA-incapable LSRs R2 and R3, the forwarding label is popped without exposing the HBH-scoped NAS to the top.
Finally, the MNA-capable LSR R4 processes the HBH-scoped NAS and brings one LSE from below the NAS to the top as indicated in the HBH-scoped NAS.

# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}} apply to this document.

# IANA Considerations

This document requests that IANA allocates a new codepoint with the name "LSE Operations" in the "Network Action Opcodes Registry" introduced in {{?I-D.ietf-mpls-mna-hdr}}.

| MNA Opcode |  Description                      |  Reference
| ---------- |  -------------------------------- |  -------------------
|    TBA1    |  LSE Operations |  This document
{: #table_iana title="LSEOps Opcode IANA allocation."}

--- back

# Acknowledgments
{:numbered="false"}

Thanks to Stewart Bryant for the input on facilitating SMEP with a generalized POP-N network action.
