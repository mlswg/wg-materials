IETF 105 MLS Friday session 10:00-12:00
=======================================

Chairing:
   * Nick Sullivan
   * Sean Turner

Notetaker:
   * Matthew A. Miller (m&m)

# User Authentication in a Group (Britta Hale)

Eric Rescorla (EKR): I just want to understand the security
property we are striving for here.  (paraphrasing) Are we
protecting from an attacker from outside the group?

* Britta Hale (BH): The attacker is from outside of group.

Joe Hall (JH): What is an Epoch?

* BH: Represents the current version of the group key, as
defined in draft-ietf-mls-protocol.

DKG: Are users interested the QR "dance" and how often
would they need to do it?

* BH: There is some interest yes.  If we want to allow for
deniability, this can help enable that.

EKR: What guidance can we give the user?  How often?

* BH: We can give recommendation if concerned about cross-comparing
with QR codes and how to do that correctly.  If you do it once you
can verify for all previous epochs.  There is also some
work that might come out that would allow an untrusted server to
verify all epochs going forward.

JH: Not as concerned about user-level; group-level is hard.  Maybe
we should think about and not necessarily design specifically
towards it.

* BH: If we want it we should make some plans, and we could try
ratcheting the signature key.

RLB: Is the proposal to generate new factors for the key schedule?
(what is the impact on the protocol)

BH: Group-level authentication vs Signature authentication.

* "For group-level, derive one more key. Using currently existing
keys requires keeping at least two keys in memory, breaking fwd
secrecy. Identity-level requires updates to the signature keys.

RLB: What about authenticating devices?

* If we use a separate auth key, we can do cross-compare better.

* RLB/BH: We are trying to auth that we have the same view of the
group.

RLB: Related to exporters -- we could do something to "off-ramp"
and escape from the key schedule, and would this fit in there?

BH: Yes, it could.

RLB: Might be good to write this up as its own I-D.  But, whether
this is a PR or and I-D this seems useful.


# draft-ietf-mls-protocol (Richard Barnes)

Benjamin Berdouche (BB): (re:consuming secrets)
We throw away the secret as soon as we derive the secret.

RLB: We should clarify that in the document.

Riab Wahby (RW): Would it make sense to make look like an API that
returns the next value?

RLB: It could be, but need to figure out how to write that down.

BB: This is exactly what we do in the security proof.

Sean Turner (ST): Some of this setup was to mimic TLS with
implementation experience, is that proving out?

* RLB: The implementations are behind the current revisions (-07).

* BB: Mine is slightly behind the current revision (-07).

* RLB: How are they verified?

* BB: The idea is that there are two implementations: one is
recursive and compact and we have multiple 1000 lines of code
in C (this one is further away).  The company one can do the
algorithms p255 and 25519.

* RLB: Maybe we get a -08 and "pause" as part of the interim to
give implementations time to catch up.


## Laziness (Richard Barnes)

EKR: How does this work with the encrypted handshake messages?

* RLB: I had not thought about it.  Part of this notion might
be sub-epochs.

JH: Does this address the group add question?

* RLB: There's a corollary < peek at slide 14> (if group changed,
MUST always send ratchet before next app msg).

Raphael Robert (RR): In the second line, there's a box missing
about blanking, and that impacts the ratcheting steps.

JH: If you have a group is mostly "silent", that "first message"
is going to very expensive?

* RLB: Yes, that's a good point.  Trade-off between when you ratchet
vs when you update.

* RR: For context, wanted to defer hard computation work.  Could
avoid expense by doing ratches more.

DKG: Question: Can you elaborate on "Add*"?

* RLB: The asterisk is that someone outside can add into group,
but cannot provision new secrets for the group.

DKG: Concern: Not sure server-initiated is worthwhile, and if we
can prevent that we have a better protocol.

* RLB: To put bounds on it, we still have strong authentication and
authorization provisions here; something could go unnoticed, but the
cryptographic properties are maintained.

EKR: The server-initiated rationale was to allow new devices to be
added without access to older devices owned by the same user.

* RLB: The other implementation is to allow the server to send a
request, that a member completes.

* EKR: The risk is server-side key escrow.

* EKR: Is there a material security difference between
server-initiated or server-requested?

* RLB: The server-initiated is in-protocol and more accountability.

* This is covered better in the next topic.

RW: Do you have a sense of the range of optimality points
(very-lazy vs not-lazy)?

* RLB: It looks like there's enough variability that we can't have a
single frequency.

RLB: Excepting the server-initiated, it sounds like this is OK with
the WG.


## server-initiated add

DKG: This hasn't changed my concern on server-imitated concerns.

* RLB/DKG: Not sure server-add has more accountability than
in-band server-invite (though less with out-of-band).

* DKG: Not clear what recourse others users have here other than
A regular message that says "RLB got a new device".

Jan Metzke (JM): I see a real benefit for something like this
in-protocol, lots of usecases for it.  Better to have something
defined otherwise it would get abused because there are lots of
usecases for it.

* RLB: If you include the solicitation in the transcript you have
similar accountability. Server-solicited add doesn't have good
(other-client) offline properties.

BH: Really see the usecases for server-initiated, but the limits
on quantum-safe options is very worrisome.

* RLB: We should see if we can find a way that this is still working
with KEMs instead of DHs.

EKR: Going back to first principles, this looks like something we
should add at the application level, but have three options.

* Option 1: Don't bother.

* Option 2: Build constructs that tacitly permit but don't outright
support (e.g., verify/audit the work).

* Option 3: Build in the ability.

EKR: Counter argument to lack of server-initiated: Are you better
off in a world where others must add, or you can self-ignore the
adds.

* Live in a client-server world, with lots of clients per person.

* If you force one path, then we don't get cross-fires and the
chance of screwing up the tree.

* Trying to avoid the case where one screwed up client messes up
the group.

JH: I just want to be suer we support the case where all adds are
confirmed by participants.

RLB: Were never trying to stop that user mode.

RLB: Will take another shot at this.


## Which HPKE (Benjamin Berdouche)

RLB: The short answer (to adding context) is we should
put in the symmetric inputs (cheap), but not the extra
PKI initiator (expensive).


## Non-destructive Add (Benjamin Berdouche)

RLB: I think we should just do this.

DKG: Did not fully understand; what happens with 3 adds? 4 adds?

* RLB: You have a private key known pre-add, and a list of leaf keys
added.

* DKG: Doesn't this make add more efficient but send far less efficient?

* BB: You only need to send to the tree + unmerged users.

M&M: Less concerned about frequency, but this looks a lot more
efficient for very large trees with (typically) few adds.

RR: This is very interesting, but concerned about the complexity.
This is orthogonal to laziness.


## Tree Signing (Benjamin Berdouche)

EKR: The (question mark) nodes are wrong (i.e., liars)? I'm not
sure I'm concerned in wrong trees but right leaves.  I care about
confirm the partition.

EKR: (in other words) I'd like to be able to tell you created the
partition and can confirm the modifier.

DKG: Which group members?  Including the ones where you've been
lied to?

EKR: As a sysop, I want to correct as quickly as possible so I need
to know whether encrypted messages are going to the right members.

DKG: Not sure what these extra signatures concretely describe, and
worry the transplanted signatures, i.e., somebody grafting subtree
onto somewhere else.

* BB: We want to confirm that e.g., a updated the tree, and others
can verify.

* BB: There is no way to enforce the same secrets on all states.

DKG: Not concerned about forging, but about replay.

* RLB: We have an id for the group.

* BB: All the participants of the people in the subtree are mixed
into the group.

RLB: To wrap up, EKR's approach might be more practical compared
to this, i.e., a state confirmation approach.


# Next Interim (Chairs)

Chairs: Sometime in September

BB: Is one day enough?  Maybe it should be 1.5 or even 2 days

* Chairs: Will get consensus from the WG, depends on number of
topics and presenters' timeliness.

BH: CaN you do the poll now?

* Chairs: Poll will go out today or this weekend.
