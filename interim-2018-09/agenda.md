# IETF MLS Working Group Interim Meeting

**NOTE**: Attendees (remote or onsite) must register; see the meeting arrangements page.

* [Meeting arrangements](https://github.com/quicwg/wg-materials/blob/master/interim-18-09/README.md)
* [MLS Architecture Draft](https://github.com/mlswg/mls-architecture)
* [MLS Protocol Draft](https://github.com/mlswg/mls-protocol)
* Chat: mls at jabber.ietf.org

## MLS Dashboard

### Architecture Updates
No updates to this document since the first revision. Need reviewers to give it a read, see how well it represents current thinking, e.g.:
* What services the server provides, e.g., sequencing
* Exactly what information is exposed to the server
* Chairs to recruit some reviewers

### ART vs. TreeKEM
The current spec is ambivalent between the two.\
#### Pro ART:
* Smaller messages by ~2X
* Doesn’t need an encryption primitive (cf. Handshake Encryption)
#### Pro TreeKEM:
* Mergeable updates (but we said we’ll assume ordering)
* O(1) processing by handshake message recipients
Are we ready to commit to TreeKEM?

### Authentication
We need an authentication layer here.
Handshake messages and content messages.
PKI or identity keys or rotating identity keys.
Barnes presented an outline at IETF 102, but it needs to be made concrete.

### Message Protection
Beurdouche presented some proposals at IETF 102.\
Need to update / finalize PR.\

### Handshake Encryption
DKG noted at IETF 102 that it would be good if the server could not tell what handshake messages are being sent, or distinguish handshake messages from content messages. \
We would obviously need to define an encryption scheme.\
Syntax\
What keys are used in what circumstances.\
There are also some larger architectural implications.\
UserAdd is basically impossible, at least in any way that authenticates the roster.\
If we want indistinguishability, sequencing requirements may extend to content.\
Need a worked-out proposal to evaluate trade-offs, decide what is worth doing.\

### Remove without double-join
Barnes proposed on list a way to use TreeKEM to do remove without causing a double-join.\
On the one hand, avoid double-join\
On the other hand, fracture the tree / more tree math\
Which trade-offs do people prefer?\

### IETF 103 Hackathon Planning
What parts of the spec are firming up enough to where we can try to get interop?\
Who is willing to work on stacks?\
What interop testing would we be able to do?\
Test vectors\
Passing live messages\
Should we develop a framework, e.g., a standard CLI interface?\

### Formal Analysis
What parts of the spec are firming up enough to start modeling?\
Where is more specificity required?\
Who is working on modeling?\
How can the rest of us help?
