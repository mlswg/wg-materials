# Thanks, in advance, for whoever volunteers to take notes!

# MLS@IETF103 2018-10-05 @ 16:10 UTC+07:00

- Two implementations at hackathon -- Wire and mlspp
- Crypto interop issue

## TreeKEM: double-join

DKG: How do you get into the state where the root node knowns but one of 
the children are not?

RLB: Adds, updates, and removes. Adds leave leaf-to-root path empty, 
removes remove the path from leaf to root, and updates heal the path.

EKR: Agree that Init state is terrible, but all protocols have a linear 
setup phase.

Ben: Don't you also expect many group members to send update right away.

RLB: Yes, in practice, and we're designing for asynchronous behavior. If 
half the group never comes online, then half the tree is never 
populated.

RLB: We need to have the Book information available and serializeable to 
inform people of group changes.

EKR: Is this is all a terrible idea?

RLB: Yes, outside of this one use case.

DKG: What is the x-axis here?

Raphael: X-axis is the time of people coming online and doing an update.

RLB: No adds or removes here, and there are more time steps here. 
(Previous chart was first 50 or so, this is first 1000.)

EKR: How much simpler does life get if Creator is special and cannot be 
evicted.

Raphael: Gets cheaper if you don't get clean the top of the tree. Cost 
seems reasonable while still being able to evict the Creator.

RLB: If you want double join at Init time and evict the creator, then 
you need one bit per node in the tree to indicate if the creator is 
double joined.

EKR: Recommendation is creating tree and then giving everyone 
bookkeeping information to evict the Creator.

Ben: If you allow creator to be double joined but not evicted, there's 
no extra state.

RLB: If you prepopulate the whole tree, you're never *not* log(N) (no 
warm up period). You don't have to do any additional DH operations 
because you already give them the secrets.

DKG: Discussion is taking place based on assumption that large groups of 
people will be added (?)

Raphael: Can probably do some simulations to determine if prepopulating 
is better than Init.

RLB: Sending messages is constant time. Adds are easy. Updates and 
removes are expensive since information goes to subsets of the tree.

EKR: Prefer optimizing the joining operation in favor of expensive 
preopulating costs.

DKG: (missed)

EKR: Two sorts of models: (1) bulks adds and (2) spontaneous adds and 
slow growth. Let's keep things simple.

DKG: Predictablility of management might be easier or better in some 
cases.

RLB: Do Init with full tree warmup.

Raphael: Agree.

Ben: How much more work would it be to do a new Init for a new tree?

Raphael: Depends on how many people are in the tree. And you destroy 
everyone's local state by doing so.

Mark: Trees can get quite badly unbalanced?

Raphael: If empty nodes are skewed (?), yes.

## Authentication

<nada>

## Hierarchical key derivation

DKG: Considered usability implications of having the phone online in 
order to start new chats. Could the phone vend not-yet-used HDK roots to 
start new chats?

Nadim: Trying to find middle ground between Signal and WhatsApp.

DKG: Recovery from compromise is the goal here.

Nadim: Could new HDK keys be derived each epoch, or would that 
completely destroy usability?

DKG: Requires a lot of focus on usability.

Emad: Disagree with Signal's usability decision over WhatsApp. Current 
draft assumes that multiple devices use multiple identity keys.

Nadim: Only comparing properties of different systems, not trying to 
argue for superiority.

RLB: Right now the spec only talks about identity keys within the scope 
of a conversation. You might have a separate identity across 
conversations. Manual key verification regimes might just use one 
identity key across conversations. Might consider comparing against 
continually re-signing identity keys.

Emad: Mixing concept of identity and signing keys. Signing keys are 
always changing. Identity keys are long-term keys.

Jon: How does this compare against a situation where every device has 
its own key, but every QR code and sign key has the "master" device sign 
an identity key of a new device. One device has the master identity key, 
other devices have generated identity keys.

## Handshake encryption

RLB: Clarifying that the cost is data transfer cost, not DH operations.

DKG: We are assuming separate identities per device, and cost is 
operations between devices related to the same identity (?)

Ben: Chart is showing that Welcome HS linear in number of group 
messages. We already accepted linear state in terms of roster and 
identity keys. How worried do we need to be about this?

Raphael: Trying to make it clear that this exists.

DKG: Current mechanism in getting state to new user is through 
distribution server. Nothing in spec that says it cannot pass through 
distribution server. So "should we allow OOB transfer of state" seems 
odd. Are we saying out-of-order?

Raphael: (missed)

EKR: Sending message needs the root, reading messages needs the whole 
path (?) Would be useful to lay out what parts of the tree are needed 
for each operation, and which parts of the tree belong to whom.

Jonathan: In practice, no one will verify conversations. They will 
simply try to read messages (?).

RLB: Can we come up with a hashing scheme that can be selectively 
updated?

DKG: Throwing in towel in metadata protection now is premature. No 
handshake encryption seems to be that. Should there be some cutoff 
between the modes so that some groups can continue to have better 
protections?

MT: Should endpoints reject messages (under some condition)?

RLB: Yes, with a NACK scheme.

MT: Could server also use epochs to reject?

RLB: Need some consistency.

Raphael: How much identifiable information does the server really see 
with handshake encryption?

# MLS@IETF103 2018-10-08 @ 11:20 UTC+07:00

Mintues go here.
