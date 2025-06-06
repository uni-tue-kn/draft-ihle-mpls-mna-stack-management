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
  IhMe25-2:
    -: ihme25-2
    title: Reduced Hardware Requirements for MPLS Network Actions Through Stack Management
    author:
      -
        ins: F. Ihle
        name: Fabian Ihle
        org: University of Tuebingen
      -
        ins: M. Menth
        name: Michael Menth
        org: University of Tuebingen
    date: 2025-06-06
    ann: Accepted for publication in ACM/IRTF Applied Networking Research Workshop.

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
With the MOVE-N-LSE operation, a number of N LSEs from below the NAS is brought to the top-of-stack.
This can be leveraged to enable more efficient MPLS processing with MNA.
An example for this is given in this document using a mechanism called HBH preservation.
With the POP-N-LSE operation, a number of N LSEs below the NAS is popped.
This can be used to facilitate mechanisms such as Stateless MNA-based Egress Protection (SMEP).
SMEP provides an alternative to the rerouting mechanism defined for the PLR in {{?RFC8679}} allowing the PLR to be stateless by carrying Bypass MPLS Labels (BML) in the stack.

However, the stack management operations are not tied to specific use cases and may also be applied for various other use cases.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Abbreviations
This document makes use of the terms defined in {{?I-D.ietf-mpls-mna-hdr}}.

Further abbreviations used in this document:

| Abbreviation | Meaning                               | Reference                                             |
| ------------ | ------------------------------------- | ----------------------------------------------------- |
| BML          | Bypass MPLS Label                     | {{?I-D.ihle-mpls-mna-stateless-egress-protection-00}} |
| SMEP         | Stateless MNA-based Egress Protection | {{?I-D.ihle-mpls-mna-stateless-egress-protection-00}} |
{: #table_abbrev title="Abbreviations."}

# Stack Management Operations
This section describes the stack management network action encoding and the processing in an LSR.

## Network Action Encoding
The network action for stack management operations is encoded as follows:

- Network action indication: The stack management network action is indicated by opcode TBA1.

- Format: The stack management network action MUST be encoded using a Format B or a Format C LSE as defined in {{?I-D.ietf-mpls-mna-hdr}}, see {{fig-lseops-encoding-b}} and {{fig-lseops-encoding-c}}.

- Scope: The stack management network action can be used in any scope, i.e., in the select, HBH, and I2E scope. The scope depends on the use case.

- Ancillary data: The stack management network action requires four bits of in-stack ancillary data per operation (pop and move) to encode the number of labels, i.e., the numbers `MOVE-N` and `POP-N`.
   - In Format B, the four least-significant bits of the ancillary data field contain the value of `MOVE-N`.
   The next four bits contain the value of `POP-N`.
   All other ancillary data bits MUST be set to zero and are reserved for future use.
   - In Format C, the same encoding of POP-N-LSEs and MOVE-N-LSEs applies to the first data field of the LSE.
   The second data field, i.e., the 4-bit wide ancillary data field, MUST be set to zero and is reserved for future use.

- Post-stack data: None.

{{fig-lseops-encoding-b}} and {{fig-lseops-encoding-c}} illustrate the stack management operation encoding in Format B and C.

~~~~
{::include ./drawings/lseops-encoding-b.txt}
~~~~
{: #fig-lseops-encoding-b title="MNA for Stack Management Operations in Format B."}

~~~~
{::include ./drawings/lseops-encoding-c.txt}
~~~~
{: #fig-lseops-encoding-c title="MNA for Stack Management Operations in Format C."}

## Processing

A node that processes the stack management network action MUST perform the MOVE-N-LSE and POP-N-LSE operation according to the corresponding parameter `N > 0`.
If `N = 0`, the corresponding operation is not applied.

- For `MOVE-N > 0`, `MOVE-N` LSEs below the NAS are brought to the top-of-stack.
- For `POP-N > 0`, `POP-N` LSEs below the NAS are popped.

The ingress LER MUST ensure that no invalid state, e.g., a MOVE-N-LSE operation with `MOVE-N > #LSEs below the NAS`, results from the processing of the stack management network action.

If multiple NAS with the stack management network action are present in the stack and are processed by a single node, all values for `MOVE-N` or `POP-N` are summed up respectively.
The operation, i.e., pop or move, is then applied based on the summed up value.

# Use Cases and Examples
This section explains one use case for each operation and illustrates it in an example.

## MOVE-N-LSE
First, we describe the concept of HBH preservation.
Then, we explain how the MOVE-N-LSE operation is used for this concept, and give an example.

### HBH Preservation
In the MNA framework, each node must be able to access the HBH-scoped Network Action Substack (NAS).
The HBH-scoped NAS must be within the readable label depth (RLD) of each node.
To achieve this, redundant copies of the NAS are placed at different positions in the MPLS stack {{?I-D.ietf-mpls-mna-hdr}}.
This increases the overall stack size.
Further, routers must scan through unrelated stack entries to find the HBH-scoped NAS.
This adds parsing overhead and is difficult on hardware with limited RLD.

The HBH preservation mechanism keeps the HBH-scoped NAS within reach of all nodes by placing the NAS always below the top-of-stack label.
When a node pops the top label, it moves the next forwarding label below the NAS to the top.
This prevents the NAS from being exposed and removed.
Nodes can process it without scanning through unrelated labels.
This avoids redundant copies and reduces RLD pressure {{IhMe25-2}}.

### MOVE-N-LSE Operation for HBH Preservation
The stack management network action with the MOVE-N-LSE operation is added to the HBH-scoped NAS with `MOVE-N = 1` to apply the HBH preservation mechanism.
Each node popping the top-of-stack label and processing this action brings one LSE from below the NAS to the top.
This prevents the HBH-scoped NAS from being exposed to the top.

However, this mechanism poses a challenge when there are MNA-incapable nodes on the path.
An MNA-incapable node pops the top-of-stack forwarding label and exposes the HBH-scoped NAS to the top, leading to packet drop at the next hop.
To fix this, the MOVE-N-LSE operation can be leveraged to bring LSEs at the preceding node of an MNA-incapable node to the top using a select-scoped NAS.
This way, forwarding labels for upcoming MNA-incapable nodes are brought to the top and the HBH-scoped NAS is never exposed.
The MNA-capabilities are signaled to the ingress LER.
Therefore, the ingress LER places a select-scoped network action for the preceding node of MNA-incapable nodes in the MPLS stack.
No information about MNA capabilities of neighboring nodes is required in a transit node as all node capabilities are signaled to the ingress LER.

### Example
An example is illustrated in {{fig-hbh-preservation}}.

~~~~
{::include ./drawings/hbh-preservation.txt}
~~~~
{: #fig-hbh-preservation title="Example using the MOVE-N-LSE operation."}

In {{fig-hbh-preservation}}, a MNA-capable node is followed by two MNA-incapable nodes.
Therefore, the stack management network action with `MOVE-N = 2` for the MOVE-N-LSE operation is pushed by the ingress LER as a select-scoped NAS for the LSR R1.
Further, to enable the HBH preservation mechanism, the stack management action with `MOVE-N = 1` for the MOVE-N-LSEs operation is added to the HBH-scoped NAS.
R1 brings three LSEs to the top-of-stack, two from the MOVE-N-LSE operation in the select-scoped NAS and one from the HBH-scoped NAS.
At the MNA-incapable LSRs R2 and R3, the forwarding label is popped without exposing the NAS to the top.
Finally, the MNA-capable LSR R4 processes the HBH-scoped NAS and brings one LSE from below the NAS to the top.

## POP-N-LSE
First, we describe the concept of Stateless MNA-based Egress Protection (SMEP).
Then, we explain how the POP-N-LSE operation is used for this concept, and give an example.

### Stateless MNA-based Egress Protection (SMEP)
The egress protection framework defined in {{?RFC8679}} is comprehensive.
It provides a mechanism for rerouting traffic in the event of an egress failure, and explains how rerouted services and their associated context can be restored.
Stateless MNA-based Egress Protection (SMEP) provides an alternative to the rerouting mechanism defined for the PLR, allowing the PLR to be stateless {{?I-D.ihle-mpls-mna-stateless-egress-protection-00}}.
Thus, the PLR does not need to maintain a table that maps transport tunnels to backup paths.
Likewise, the PLR is not involved in the signaling of such information.
Instead, this information is supplied from the ingress to the PLR in the network action.
Signaling is only needed between ingress, egress, and the protector, but not with the PLR anymore.

However, in the proposed SMEP network action encoding in {{?I-D.ihle-mpls-mna-stateless-egress-protection-00}}, the 20 bit BMLs are split across the AD fields of a network action.
This requires the PLR to extract the BML from the network action.

### POP-N-LSE Operation for SMEP
The POP-N-LSE operation simplifies the application of SMEP.
The BMLs are contained as forwarding LSEs in the MPLS stack without the need to encode the bypass tunnel information in the AD fields of a network action.
The processing at the PLR is as follows:

1. If the PLR does not detect an egress failure
   - The PLR executes the POP-N-LSE action and pops all BMLs.
   - The packet is forwarded as usual to the egress node.
2. If the PLR detects an egress failure
   - The POP-N-LSE action is ignored and is popped along with the top-of-stack label.
   - The BML is now the top-of-stack label. The packet is forwarded to the protector based on the BML.

### Example
An example of the POP-N-LSE Operation for SMEP is given in {{fig-smep}}.

~~~~
{::include ./drawings/smep.txt}
~~~~
{: #fig-smep title="Example using the POP-N-LSE operation for SMEP."}

The example network contains an LSP R1-R2-R3 and a bypass tunnel R2-R3'-R3''.
The label stack contains the forwarding labels L1, L2, L3, L3', and L3''.
If there is no egress failure, the LSR R2 executes the POP-N-LSE action with `POP-N = 2` and pops the BMLs L3' and L3''.
The packet is forwarded as usual according to the top-of-stack label L3.
If the LSR R2 detects an egress failure it becomes the PLR.
The POP-N-LSE action is ignored and the NAS is popped along with the top-of-stack label.
This time, the BMLs L3' and L3'' are at the top-of-stack.
The packet is forwarded according to those labels to the alternative egress node R3''.

# Implementation Status

\[Note to the RFC Editor - remove this section before publication, as well as remove the reference to {{?rfc7942}}\]

This section records the status of known implementations of the protocol defined by this specification at the time of posting of this Internet-Draft, and is based on a proposal described in {{?rfc7942}}.
The description of implementations in this section is intended to assist the IETF in its decision processes in progressing drafts to RFCs.
Please note that the listing of any individual implementation here does not imply endorsement by the IETF.
Furthermore, no effort has been spent to verify the information presented here that was supplied by IETF contributors.
This is not intended as, and must not be construed to be, a catalog of available implementations or their features. Readers are advised to note that other implementations may exist.

## University of Tuebingen Implementation

The stack management network action defined in this document has been implemented using the MOVE-N-LSE operation for HBH preservation with a P4 pipeline {{IhMe25-2}}.
The implementation code can be found at [https://github.com/uni-tue-kn/P4-MNA](https://github.com/uni-tue-kn/P4-MNA).


# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}} apply to this document.

# IANA Considerations

This document requests that IANA allocates a new codepoint with the name "Stack Management" in the "Network Action Opcodes Registry" introduced in {{?I-D.ietf-mpls-mna-hdr}}.

| MNA Opcode | Description      | Reference     |
| ---------- | ---------------- | ------------- |
| TBA1       | Stack Management | This document |
{: #table_iana title="Stack Management Opcode IANA allocation."}

--- back

# Acknowledgments
{:numbered="false"}

Thanks to Stewart Bryant for the input on facilitating SMEP with a generalized POP-N network action.
