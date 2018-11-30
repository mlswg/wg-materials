# IETF MLS Working Group Interim Meeting

**NOTE**: Attendees (remote or onsite) must register; see the meeting arrangements page.

* [Meeting arrangements](https://github.com/mlswg/wg-materials/tree/master/interim-2018-09)
* [MLS Architecture Draft](https://github.com/mlswg/mls-architecture)
* [MLS Protocol Draft](https://github.com/mlswg/mls-protocol)
* Chat: mls at jabber.ietf.org
* [Webex details](https://github.com/mlswg/wg-materials/tree/master/interim-2018-09#remote-participation)
* [Bluesheets](https://www.ietf.org/proceedings/interim-2018-mls-01/bluesheets/bluesheets-interim-2018-mls-01-201809270900-00.pdf)

## MLS Dashboard

### Ways of Working

How we want to do PRs.

### Architecture Updates
_2018-08-27_ \
No updates to this document since the first revision. Need reviewers to give it a read, see how well it represents current thinking, e.g.:
* What services the server provides, e.g., sequencing
* Exactly what information is exposed to the server

**AIs From MLS@102**\
Based on -mls-archtiecture draft discussion
* Authors will add min/max recommended guidance on key updates.
* Authors will await further discussion on whether the group membership server is authoritative.
* Authors will add a reserve space for traffic analysis mitigation without details.

Based on Handshake message ordering / server trust point discussion
* Authors should add this to the -mls-architecture document.
* Authors should add the server enforces a delivery order, and provision for clients to detect ordering.
* Authors should add discussion on starvation and guidance for servers in the face of malicious clients.

Chairs to recruit some reviewers!

### ART vs. TreeKEM
_2018-08-27_ \
The current spec is ambivalent between the two.
#### Pro ART:
* Smaller messages by ~2X
* Doesn’t need an encryption primitive (cf. Handshake Encryption)
#### Pro TreeKEM:
* Mergeable updates (but we said we’ll assume ordering)
* O(1) processing by handshake message recipients

_Are we ready to commit to TreeKEM?_

_2018-09-27_ \
Slides: https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/TreeKEM.pdf

#### Remove without double-join
_2018-08-27_ \
Barnes proposed on list a way to use TreeKEM to do remove without causing a double-join.\
On the one hand, avoid double-join\
On the other hand, fracture the tree / more tree math\
_Which trade-offs do people prefer?_

_2018-09-27_ \
Slides: https://docs.google.com/presentation/d/1CoPfOEbTHfSuYb4zzdCeseEBTJRQVb90JmtvjaljBVM

### Authentication
_2018-08-27_ \
We need an authentication layer here.
* Handshake messages and content messages.
* PKI or identity keys or rotating identity keys.

Barnes presented an outline at IETF 102, but it needs to be made concrete.

_2018-09-27_ \
Slides: https://docs.google.com/presentation/d/1-f7JjzxCCrF1gngwI0bDAAD4Vo5eYVsVuAUrk7hckCA


### Message Protection
_2018-08-27_ \
Beurdouche presented some proposals at IETF 102.\
Need to update / finalize PR.

_2018-09-27_ \
Slides: https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/2018_09%20Interim%20-%20MLS%20Message%20Protection.pdf

**AIs from 103@IETF102**

Authors to discuss amongst themselves about next steps

### Handshake Encryption
_2018-08-27_ \
DKG noted at IETF 102 that it would be good if the server could not tell what handshake messages are being sent, or distinguish handshake messages from content messages.

We would obviously need to define an encryption scheme:
* Syntax
* What keys are used in what circumstances

There are also some larger architectural implications:
* UserAdd is basically impossible, at least in any way that authenticates the roster.
* If we want indistinguishability, sequencing requirements may extend to content.\
Need a worked-out proposal to evaluate trade-offs, decide what is worth doing.


### IETF 103 Hackathon Planning
_2018-08-27_ \
What parts of the spec are firming up enough to where we can try to get interop?\
Who is willing to work on stacks?\
What interop testing would we be able to do?
* Test vectors
* Passing live messages
* Should we develop a framework, e.g., a standard CLI interface?

### Formal Analysis
_2018-08-27_ \
What parts of the spec are firming up enough to start modeling?\
Where is more specificity required?\
Who is working on modeling?\
_How can the rest of us help?_

_2018-09-28_ \
Slides: https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/2018_09%20Interim%20-%20AuthMLS.pdf
