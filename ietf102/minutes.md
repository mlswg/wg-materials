MLS @ IETF 102
2018-07-29 @ 09:30 EDT

================================

# 0. 10min Administrivia (Chairs)

No actions.

# 1. 05min Charter Review (10Kft level)
        https://datatracker.ietf.org/group/mls/about/

No actions.

# 2. 10min Architecture (Emad)
        https://datatracker.ietf.org/doc/draft-barnes-mls-protocol/

rlb: Yes, the draft should provide guidance on key update defaults, but
allow flexibility.

Philip P: Shouldn't be based on the strength of keys?

Raphael Robert: I don't think the server should be trusted.

Benjamin Beurdouche: The suggestion to provide guidance is a good idea.

Stanislav Smyshlyaev: I totally agree we should provide guidance.

ekr: Yes, we should provide guidance on key update, and guide people on
how to think about the problem, too.
    - The server should not be trusted to manage group membership, but
it should be trusted to note group membership.
    - We should not do anything about traffic analysis mitigation here.

Jon Millican: We should we include guidance that makes sense for small
(0 - 10) and large ( 50k+) groups.

Emad: The guidance can be when you are sending versus just receiving.

Jon M: We need to be careful, in case *any* member is compromised.
However, I agree with you.

dkg: I recommend framing it as min/max guidance.
    - I think we shouldn't get too deep into multi-device for privacy
reasons.
    - We need to include a mechanism for padding, but do not have to
define specifics.

Lucia B: Having a recommendation would be useful, but a strict
requirement will hinder adoption.

Bob Moskowitz: Good guidance for different class of scenarioes would be
most useful.
    - I think group membership store is out of scope for MLS.
    - MLS needs to include traffic analysis mitigation for itself.

Karthik: It's important to ask what to expect for group membership at
this stage, should all members maintain the list of be able to query for it?

rlb: Largely +1 on other points (from key updates)
    - You might have a subtree of the devices under the overall tree of
user members, but we don't need to get into the details of device hiding
right now.
    - We might use the server to be a group membership cache, but not
require it to store the full tree (group state but no identity information)
    - Not worried about protecting key exchange messages, but with
actual message protection scheme (and we should have some mechanism left
to apps to define).

ACTIONS:
Authors will add min/max recommended guidance on key updates.
Authors will await further discussion on whether the group membership
server is authoritative.
Authors will add a reserve space for traffic analysis mitigation without
details.

Room consensus is to adopt -mls-architecture as a WG item (to be
confirmed on list).

# 3. 25min Handshake message ordering / server trust (Emad)

rlb: The architecture document should have a clear statement on handshaking.
    - If the client can rely on forced order delivery, then their
implementation becomes simpler.
    - Comfortable with server-enforced ordering, as long as don't trust
to guarantee service

emad: My concern with server trust is specific denials, but we can
mitigate that

dkg: The requirements note the client must be able to detect
out-of-order delivery.
    - I don't know what this means in a distributed messaging system.

rlb: The risk here is with handshake messages that the key messages rely
on a certain state.
    - If they are not delivered in the same order to all clients, then
the clients have an inconsistent state.
    - The goal is for clients to agree on the cryptographic state of the
group membership.

ekr: The system can't guarantee order with simultaneous updates from two
different clients.
    - We can guarantee messages from a given client are delivered in the
same order.
    - There are limits to what is possible, but we should be able to
detect suppressed or reordered messages.

Raphael Robert: This should be part of the -mls-architecture document.
    - We need a way for clients to depend on the order without relying
on the server, and that is difficult.
    - The ordering requirement comes down to ART vs. TreeKEM, and the
best solution is to trust the server to enforce ordering.
    - I don't think starvation is a problem if just worried about
handshake messages.

Jonathan Lennox: The problem with starvation is genuine overload vs a
malicious server, and you should be able to detect that difference.

Bob: Where's the state machine here?  We should have the state machine
before we can talk about ordering.
    - Starvation is DoS attacks, and we should understand that.

Jon Millican: It is desireable if the server could do nothing to
compromise security short of denial of delivery.

rlb: The order that is important to preserve is the order that messages
are delivered; two-phase commit is useful there.
    - I think it's worth adding a section on starvation, and the server
can help for client-side issues.
    - I'm surprised federation hasn't come up with this ordering
requirement.

daniel franke: We can't absolute order, but we can use things like
vector clocks to track relative ordering.

michael motavsky: Messages specific to state are important to preserve
ordering and maybe bound cryptographically, but others it doesn't matter
as much.  We should separate those more.

Leif Johansson: Thinking about the federated case, would it make sense
to indicate to the user whether or not there is message order
contention.

michael motavsky: There's not much you can do with respect to
starvation, you might want to create a mechanism to allow the
client to back out until a recovery mechanism is in place if they were
unable to participate for a period.

Brian Weiss: We are trying to trust clients (or not trust anyone) but
not servers, and that's not clear in these requirements.  That should be
clarified.

Jon Millican: With locked-out clients, the protocol might be able to
automatically expire clients somehow.

rlb: We should have consideration on how to detect starvation that we
can address when appropriate.  We should docuement it can happen, and
what the server can do to help in the face of malicious clients.

ACTIONS:
Authors should add this to the -mls-architecture document.
Authors should add the server enforces a delivery order, and provision
for clients to detect ordering.
Authors should add discussion on starvation and guidance for servers in
the face of malicious clients.

# 4. 25min ART vs. TreeKEM vs. both (Jon)

rlb: What do we need for right now to move forward: which of these do we
want to keep?  Right now it's "use either", but that's not ideal.  They
are mostly interchangeable, so maybe we can pick one for now and swap
later after further analysis.
    - I don't think there's a strong winner here.

Katriel: I would like to hear on-list from pro-federation people on
concurrency concerns.

Stansilav: We really need to analyze TreeKEM properties, and CFRG can
help with that.
    - Both constructions are so similar we can get away with either or
both, but I prefer ART's construction

Benjamin Beurdocuche: We should keep both in the spec for now.  Both
need more analysis, and they are so similar.

Karthik: There are a lot of purposeful commonalities to keep other open
questions orthoganal to the ratchet mechanism.
    - We hope to address the analysis question in the coming moths.

rlb: Despite the requirement for consistency, it doesn't require
blockchain serialization.
    - It's important to understand the lifetime of an epoch, and when it
is ok to discard for a later one.
    - The delivery consistency makes this a little easier, but it
doesn't necessarily mean TreeKEM is better here.
    - We should operate under the assumption both are secure, and work
and pick the one that is better operationally.
    - Having two makes life hard.

Ben Kaduk: Don't send the document to the IESG with both, pick one
before submitting!

Raphael Robert: ART vs TreeKEM has a big impact on passive members, and
has impact on key update guidance.
    - ART is expensive for passive members to calculate the group key
every update, that TreeKEM does not require.

rlb: With regard to epochs, we have the problem for self updates as well
as group updates.
    - We need to have a convergence algorithm to reach a consistent state.
    - Recommend keeping things ambiguous for now to work it out in a
"hackathon" in Sept.
    - Maybe we cross the trees: If we think updates are super-frequent,
we use TreeKEM to keep them efficient and ART for adds/removes?
    - The keys are derived differently, but they are still public keys.

Katriel: /facepalm

Shivan: If we go with both options, then we detail the pros/cons better,
either here or in a separate draft.

Emad:  There is still O(n*log(n)) for public key ops (for the actor
only, though).

Actions: No actions



# 5. 25min Message Protection (Benjamin w/ local support from Richard)

ekr: What does the "Weakened Forward Secrecy (if you never send)" mean?
 The key is never used ...

Benjamin B: The keys are kept for a long time.

ekr: If the group is always ratcheting, then keeping around an unused
key is not an actual risk.

rlb: I agree we should get something in the -mls-protocol document.
    - Comparing the key schedule, the "killer app" is the reduced
storage (favoring group ratcheting)
    - Not sure the ordering will be a big problem.
    - We should keep one of these and swap it out if needed.
    - I recommend the signature be mandatory.

Benjamin B: I would like the signature for handshake messages, but maybe
not for content messages.

rib: I think we should have it as the default.

dkg: I also don't see the forward secrecy advantages for group
ratcheting, and sender ratcheting is the clear winner.

ekr: I'm concerned about the window of state a receiver needs to keep to
deal with reordering.

-- missed (minute taker in mic line) --

emad: Per-participant makes more sense.

Jon Macillican: I'm concerned that we build for low-end devices as well
as iPhone X (-:

Jonathan Lennox: If you want per-participant signatures, you'll need
that key which will be **much** larger that these keys.

ACTIONS:

Authors to discuss amongst themselves about next steps.

# 6. 25min Authentication (Richard)

Jon Millican: When clients have the full list, why not store the entire
tree for removes?

rlb: I think it's totally plausible clients have the entire tree, but
the hard part is initializing that tree for new clients.

Jon Millican: concerns -- It seems require the server to store more
metadata.  Why not have the server store the entire tree + commitment.

rlb: If you have the commitment of tree-over-tree, the server could
strip out identity bits (keeping less metadata), and use OOB for
identity information.

Karthik: Hard to get PCS with signature keys, because of long-lived
identity keys.  Benjamin might have a method of mitigating that, but I'm
less worried about the authentication keys vs. confidentiality keys.

Lucia Ballard: What are the authentication properties for receivers (how
do I validate long-term passive participants are still valid)?

rlb: Mostly concerned about joins; the simplistic model is to retransmit
the transcript of the changes.

Benjamin B: I don't like signing things with a long-term identity key,
but we could get better by having ephemeral signature keys.
    - For denialability if we disconnect the identity key from the group
key (??).
    - In general, this should not impede progress.

rlb: These schemes for denialability and PCS are additive, so we can
assume we have long-term identity keys for now and add short-term
signature keys later.

Stanislav Smyshlyaev: If we have long-term identity keys, then these
proposals are fine.
    - Do we want to give recommendations for users that want to be truly
anonymous (e.g., provide an OOB mechanism)?

rlb: The authentication server is mostly separate, so the credential
here could be opaque for group key operations, but means something to
the authentication server.

Stanislav Smyshlyaev: It would be very good to have recommendations to
protect privacy cases.

Raphael Robert:  The attacker might be able to compromise group key and
identity keys with local access, but the mostly separated architecture
protects from most other parallel attacks.
    - The only pratcial approach is to have optional denialabity in
order to move forward for now.

dkg: Potential user experience confusion in SIGMA because of a success
in one part but a failure in another, unless we expect "all or none" for
validation.
    - We should add guidance for what happens for partial SIGMA failures.

# 7. Working Group Actions

The consensus in the room to adopt the -mls-protocol document in the WG.

# A. 25min A.O.B

## A.1 Versioning / extensibility

## A.2 Interop testing framework

## A.3 Interim plans (Late sept, Berlin?)

To be announced.
