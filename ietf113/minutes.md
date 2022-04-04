# IETF 113 MLS WG session

## Agenda
### Administrivia

- Virtual Meeting Tips: QR code reminder; use queue tool if physically in the room
- Note Well
- Note Taker - Robin Wilton
- Jabber Scribe
- Status
    - Timeline has been structured to ensure that the protocol and arhcitecture docs have to make parallel progress towards WGLC.

### MLS Protocol - Richard Barnes

I-D: https://datatracker.ietf.org/doc/draft-ietf-mls-protocol/
- Draft 13 update: 
    - Editorial work to re-order many sections, but no technical changes (just improved flow);
    - Streamlined syntax of vectors, using same idea as CTLS and QUIC
    - Split leafnodes from keypackages; keypackages is now used only to pre-publish information about an entity. Also allows key separation to be slightly improved.
    - Algorithm for updating the tree has been updated to remove "redundant" nodes, where redundancy = having only one child at multiple subordinate layers.
    - "Parent" hashes in the hierarchy now bind more data about siblings and children of siblings - but there are some subtleties to cater for at the "right hand edge" of the tree (see slides and draft). Performance his is not devastating for most use cases, according to simulation.
- Draft 14 update:
   
Open Issues/Pull Requests
   
   - Issues
        - 619; annotation and clean-up. ACTION: Please merge for AASVG
        - 618: ACTION: merge
        - 617: some ambiguity about deniability in the spec; language has been drafted to clarify who are valid signers of a message.
        - 605: 
           - Add a MIME_type object to allow applications to express MIME type of application_data is wished. Richard Barnes' proposal is to close this on grounds that it is premature. 
           - Rohan Mahy: this is basic protocol hygiene, not premature, and would bring MLS into line with other protocols at the cost of a single byte.
           - Richard: this is analogous to ALPN as an addition to TLS.
           - EKR: Interesting analogy, but MLS is designed to be embedded in other protocols, so this function could be left to the other protocol.
           - Rohan: Sending this data outside MLS could be seen as informational leakage...
           - EKR: distinguished between defining the message type (inside MLS) and defining the format for expressing the message type (and how to inspect/treat the contents) outside MLS.
           - DKG: ... and that won't always be data about MIME types.
           - Ben Kaduk: It is good to be able to uniquely determine the structure/format of the message contents, but whether that information needs to be inside our framing or the outermost layer of framing for what we convey seems subject to debate.
           - Rohan: If your messaging group/app only ever sends one type of application data, you can safely igore this parameter or leave it blank. We're only talking about a cost of one byte, and it increases agility among data types supported.
           - Jonathan Hoyland: How is this data accessed by the application, and how would inconsistencies/mismatches be handled?
           - Cullen Jennings: [paraphrased!] It's easier to add a field to an existing wrapper than to define new wrappers for each case.
           - EKR: this is an instance of a common problem, namely how to demux (de-multiplex, segregate, distinguish...) different types of traffic in an opaque transport layer.
           - Hoyland: If the information isn't used by the MLS layer it makes no sense to me to put it in the MLS layer.
           - Kaduk: [Paraphrased] The question about where best to represent this parameter arises because here, we're talking about the structure of MLS, not the full protocols that embed MLS.
           - DKG: One solution would be to have clear documentation of how to integrate MLS into your protocol (+1 from Kaduk).
           - Rohan: [Paraphrased] The goal here would be to reduce the amount of parsing an application needs to do to determine content type (i.e. parse a defined parameter, rather than inspect the contents and infer type).
           - Kaduk: Could clearly document what the app is expected to do; defining it here risks precluding better subsequent ideas.
            
Chair: calls for a show of hands. The question will be specific to merging issue #605, raise hand for merge, do not raise to not merge.
Show of hands was 50:50, issue is therefore deferred to the list.  

GitHub: https://github.com/mlswg/mls-protocol

Security Research

### Architecture - Benjamin Beurdouche

I-D: https://datatracker.ietf.org/doc/draft-ietf-mls-architecture/

GitHub: https://github.com/mlswg/mls-architecture

Open Issues/Pull Requests

- #75 Track Expected Types of Deployment
    - Ticket in Github lists a starter set
    - ACTION: closed
- #71 Current text does not define sufficiently strongly what constitutes "being a member of the group"; as a result it does not reflect the properties of the underlying key exchange.
    -  Barnes; membership != liveness or consent
    - Wilton; sought confirmation that any ambiguity is in the text, and not reflected in loss of security/transparency in the key exchange/protocol. Beurdouche confirmed. 

### Federation - Raphael Robert

GitHub: https://github.com/mlswg/mls-federation

Resurrection plan, now that the protocol itsenf is close to WGLC, there are several implementations and a  deployment. (Barnes: Two! RingCentral uses it too!)

Chair: Draft already adopted. Editors are free to work on it with no further re-opening process.
