#  MLS Interim Minutes
Date: 14-15 January 2019
Location: Mountain View, California

Sean: [Note well](https://github.com/mlswg/wg-materials/blob/master/interim-2018-09/2018_09%20Interim%20-%20Chair%20Slides.pdf), this is an IETF meeting etc. We went around the room etc doing introductions. WG background, agenda. Sean: goals and non-goals, timeline.

# Day 1: 14 January 2019

## Morning

## Open issues: architecture document
[we went over the open issues quickly, preparing for later presentations. I've filtered
the issues where we just read over and acknowledged them.]

### Terminology
Ben: we should standardise on a single terminology for the two documents. Currently “participant” versus “client”, “member” etc. Suggestion: client is a key. We whiteboarded names for different protocol roles.

Richard: architecture should talk about various roles (users, devices, etc) but the protocol does not need to know about the role of e.g. human users.

AI(Katriel + Ben): we will go through the pull request and bring a concrete proposal to the group. Yevgeniy: I can join. We need to capture e.g. human user into the architecture. Yevgeniy: giving the precise inputs to the various algorithms (Update etc) should clarify this.

Ekr: if you have an all-keying-material-per-device model (i.e. every key is a separate user) and there’s a higher level concept that provides the way to glue multiple keys into a single user, then you need system design juggling when users switch devices.

### Review for architecture doc
AI(Ben): I volunteer to go through the architecture docs and report issues.

### Group policies
Yevgeniy: how do you say which user can add/remove/etc? Karthik: we need to know that every group management is authenticated. Ekr: we need self-add (though rlb: we don’t have it at the moment).

Katriel: perhaps we need to actually specify some policies (e.g. anyone can delete anyone else, or there are admins and non-admins). rlb: more important is highlighting where in the protocol flow you would do this. If you have encrypted handshake messages then the server can drop violating messages. Karthik: you need to make sure that every device has the same policy.

### DoS by a malicious user
A bad actor can send a malicious message that desynchronises the group. rlb++: one case is that an individual participant can detect that something has gone awry. You might be able to come up with a way for participants to detect who sent the malicious update. We’ll come back to this later.

### Retry considerations
The protocol requires that all participants see things in the same order. Clients might need some retry logic so that if they send a message and it gets bounced by the server. At minimum, the protocol operations could just fail, but we’re not sure whether the protocol needs to handle “if your update failed then retry it”.

### Encryption of Welcome Messages (rlb)
[Slides](https://github.com/mlswg/wg-materials/blob/master/interim-2019-01/2018Q1_MLS%20Interim-rlbpdf.pdf)

This is a PR that’s already landed. The change is that it now encrypts the messages to a UserInitKey instead of sending it in clear.

ekr: it uses an opaque key_id instead of hashing the key. Does a key id correspond to a family of ids? We could just take the UserInitKeys and make each UserInitKey correspond to one ciphersuite. However, raphael points out that you need to handle the case that the server doesn’t speak that one. rlb: each curve is paired with one hash algorithm and one symmetric algorithm. ekr: this isn’t sensible for symmetric algorithms.

sn3rd: why would the userinitkey be empty? rlb: if you decide only to ever publish one userinitkey then you could leave it empty.

ekr: is it permissible to publish two keys with the same id?

alex: handshake messages are not encrypted. Does it make sense to encrypt them?

### Ciphersuites

rlb: right now we have ciphersuites that identify the tuple (curve, hash, symmetric cipher). The proposal here is to have different identifier spaces for (curve) and (hash, symmetric cipher), following the pattern that TLS is doing. ekr: we could also then just “steal” TLS’s numbers.

alex: what about matching cryptographic strengths? ekr: up to you.

What about handing postquantum negotiation? Yevgeniy: it would be nice to minimise the reliance on Diffie-Hellman and use KEMs instead. rlb: this is ok as long as we’re using TreeKEM.

### Garbage collection

Right now trees only get larger, and leaves once blank are always blank. We could clean that up a bit. ekr: if you chose a random point to add people to, instead of just adding them to the right of the tree, then you might fail to bounce adds.

When you remove someone, you could also reduce the size of the roster and the tree until the interior nodes are non-null. We discussed a few algorithms for trimming the tree.

Once you have these tree-trimming operations, you can do a few things. You can Resync: remove someone and add them back in place again. You can do move: remove+add+update. You can do other stuff as well.

Karthik: it’s worrying to increase the number of cases where you have blank nodes, since they hurt performance. It would be nice to aggressively get rid of them.

Yevgeniy: instead of overwriting keys and then blanking them, can you choose a random delta so that everyone on that path adds the deltas to their existing path. Then you don’t need to blank it, you just need to not encrypt the delta to the last person. rlb: the challenge is that you have to send someone the delta but also the public keys for the updated things. Yevgeniy: with DH you might be able to do it homomorphically. Maybe some other schemes would do it.

### 2-3 trees

New proposal from Yevgeniy Dodis: TreeKEM uses binary trees. They have the problem that they’re not balanced so the worst-case performance can be bad. There are lots of balanced trees, most of which use tree-rotation which isn’t good for us because parents become children. However, there’s a simple design called a 2-3 tree which has O(log n) insert and delete. Invariants: every node has degree either two or three, and every leaf is at the same depth. This automatically guarantees depth between log3 and log2 n.

If the inserter has only one sibling then they add the new person as a direct sibling. You use the fact that 3+1 = 2+2 to rearrange if a node is full, and you might have to recurse up to the root in the worst case. Deletes are similar: if you are one of three, you just delete. If you have a sibling with degree three you do A Thing and it works but with log size in the worst case.

We discussed this proposal but did not come to any conclusions.

## afternoon

### Verifying keys for each user

Jon Callas: there's been some discussion about a server introducing keys for users which do not correspond to that actual user. We should consider this. Katriel: we could easily add something into the arch doc. ekr: in whatever we design, it should be straightforward to detect this server behaviour. JonC: some systems alert the users on new keys and some don’t (Signal is cranky about infrastructure things, WhatsApp is very quiet). Nalini: can you clarify who would detect? What’s the difference if you were adding a legitimate monitoring box? Nick: we are chartered to have membership authentication.

Nadim: why are we talking about this? JonC: because it will be raised for the standard. Nadim: the “membership authentication” requirement means we must prevent this. rlb: it isn’t fully accounted for, because they come up as an attack by the authentication service. We’ve been thinking about the authentication service as trustworthy. Nadim: what about fingerprint verification? JonC: even fingerprints have complications.

Benjamin: does it suffice to add “if you don’t trust the authentication service then you SHOULD do X?”. ekr: two things. First, if the authentication service is untrustworthy then it can insert new devices that can represent a new identity, and if we want to prevent that then you can do Y. Nick: we should make it possible to have multiple points of authentication (e.g. KT + authentications service).

JonC: note that we don’t want to notify recipients, since they are going to just click through warnings. We want to notify senders that new devices have been added to their accounts. ekr: this is what CT does, effectively.

AI(Katriel + ekr + JonC): write an issue and a pull request to add something to the architecture doc.

### Proxy Re-encryption

[Slides](https://github.com/mlswg/wg-materials/blob/master/interim-2019-01/2018Q1_MLS%20Interim-RR-PRE-00.pdf)

Raphael presented an idea for using proxy re-encryption to encrypt metadata that the server stores for clients.

Karthik: if you’re only serving ephemeral keys there isn’t that much metadata. Can’t you just store them in plaintext? Raphael: yes, for just public keys, but what about the roster? Jon: but you need the roster in most apps, especially if you have fan-out. Raphael: yes, but fan-out is ephemeral: you can forget it afterwards. Storing the roster permanently is different.

We discussed what metadata would be leaked under various threat models.  Raphael: reminder, this is just a proposal with some modern cryptography. Benjamin: what do people thing about PRE for this kind of stuff? Is this something we should be exploring or should we not bother?

Karthik: what is the desired property? The server should maintain some property in a way that it doesn’t learn stuff, but that clients can pull it with confidentiality and integrity.

Nadim: what are the confidentiality expectations around ephemeral public keys?

JonM: what about if different leaf keys are reused in different groups?

Karthik: we should treat the roster list as a high-value secret. JonM: we might not be able to do that, because even if the server doesn’t store the roster, any given update updates a certain leaf and path and that might be correlatable. Raphael: we assume there’s a central server instance that can observe all metadata (e.g. fanout). This is completely out of scope for MLS, you’d need a mixnet or something. The scope that was proposed at the last interim was specifically data at rest that can be subpoenaed.

Nadim: should there be something explicit in the spec about roster confidentiality? Sean: section 3.1.5 in the architecture draft. Raphael: we should be a little more specific. Filed #47.

Raphael: there is one e2e messaging system that doesn’t hold metadata on the server, and that is Signal. Everyone else, as far as we know, has the group list on the server and on top of that there is no particular cryptographic assurance of who is in the group.

### Lazy handshake messages (Raphael)

[Slides](https://github.com/mlswg/wg-materials/blob/master/interim-2019-01/2018Q1_MLS%20Interim-RR-LazyUpdate-00.pdf)

We inherit the ART multidevice mode: each device had its own leaf, and adding and removing members results in multiple handshake messages. The roster is a list of devices, not members. Some logic is pushed to the application layer (knowing which devices correspond to “Richard”).

Another option is for a leaf to represent a user, and under each leaf is a subtree of devices. That tree is shielded from others in the group. In practical, if a user adds or removes a device that looks like an update handshake to the rest of the group. At least on this level, the group cannot tell if a device has been added or removed. The roster is then of the leaves which are users.

Karthik: there might be some atomic operations where this affects the protocol (e.g. add these five devices), but otherwise Raphael: it’s up to the application.

Emad: I am interested in working on the multidevice problems. JonC: me too.

Nadim: I raised hierarchical key derivation at the last interim in order to try and limit the scope of compromise. The way that Signal and WhatsApp do multidevice is different and has different tradeoffs. When we do multidevice, this might let you get better ideas around multidevice.

Yevgeniy: I like the idea of hiding your subtree. We could also use this for hierarchical MLS where e.g. Google federates with someone else, and hides the Google members.

AI(Emad, Jon): put together a document about having multiple devices under a single node

Raphael: adding a device to an account is currently the biggest pain point for us, and this isn’t a problem for most other messaging systems: you have to add it to all the groups it’s in, and you might run into the state where it’s only half added. The current cost is that you have to send an update to every group, which is O(log n) to O(n) for each group and there might be hundreds of them. Many of these groups might be dead. Tincan does the same thing: it only updates a thread when a new device is added. Right now we do everything proactively, but it sounds like we should wait until it’s actually necessary to update a group.

Raphael: in the subtree design, here’s a proposal for a lazy update. The sender creates a new leaf node but doesn’t hash it up, and the public key of the new leaf node is sent to the group in a new handshake message. Then every member in the group replaces the old leaf node with the new leaf mode, and every member blanks the sender’s direct path. This is all that happens, and it’s relatively cheap for the sender (you just generate a keypair and send it out, but nothing else happens).

We spent a while discussing different approaches to get lazy handshakes without allowing unverified users. (E.g. JonC: iMessage does pairwise links.)

Katriel: there is a real requirements change here. It seems that every messenger requires you to be able to wait for new conversations until you actually need to. UserAdd might overlap here: you could just request the next person to send to that thread to useradd the new device.

rlb: when you have a new device show up, the server knows about it and blocks existing members from sending to a thread. What about if the new device is the first to send? Is there a “KEM to the group” think that sends a request to the group that you would like to be added. As long as nobody in the group is sending, it doesn’t matter whether you’re part.

rlb and Raphael came up with a neat idea: to join a group, you just send an Add message. This makes groups invitation-only, which doesn’t sit well with general-purpose messaging. This proposes a new feature: we make a public key for the group at any time, which anyone else can use to send to the group. Raphael: if you send to this then you wouldn’t know who you are sending to. rlb: you could have a signed roster attached to the GroupInitKey.

Yevgeniy: there are two orthogonal issues. One is “how does a user outside communicate that they want to be added”. Once we have a group secret, you can optionally have stream ciphers and also a group key which may or may not provide forward secrecy.

Karthik: there is a mode in which the user being added gets a welcome packet from a group member. There’s another mode in which the user is willing to accept a welcome packet which has been signed by the server. We spent a while talking about 0-RTT guarantees, in which a new user sends data to the group without yet knowing who they are sending to: the server can lie about it. We want that device to be able to send data. Benjamin: 0-RTT is a bad bad idea.

Britta: how does your new device identify as one of your existing devices? Otherwise it could be just anyone’s.

Summaries of this: we talked about the two models of multidevice, and AI(Emad) agreed to write about the subtrees model. Then we talked about the motivation that we want to reduce the load in multidevice contexts when devices are added to or removed from an account. In that context we had the proposal of having lazy updates specifically in the subtrees mode, which turns out can be potentially be further simplified if we are happy with the fact that there is no forward secrecy between user devices as you add a new one. That means that you completely duplicate the state between devices as far as the group state is concerned, so the group does not know that a new device was added. That doesn’t work when we remove people, so there we would still have to do a lazy update because we need to introduce some freshness somewhere.

AI(Raphael): write up how to add and remove devices in multidevice. At some point we need to vote hum on how to do it.

AI(rlb) is interested in writing up 0RTT for new devices.

We also discussed UserAdds as an additional way for a user to join a group. In the first multidevice mode (1 device=1 leaf) then UserAdd could come in.

rlb: we have two functional requirements. One is when a user signs in on a new device that user can send a message to groups they are already joined to immediately. Two is that that has to occur lazily i.e. it doesn’t affect a group until there’s some other activity in that group.

Karthik: we have tried to make operations on individual groups efficient, but we have not really looked at operations across groups.

There are two modes. In one mode, devices are only added to groups when someone in the group comes online and adds them. In another mode, devices can pre-join groups and send 0-RTT.

# Day 2: 14 January 2019

### Terminology part 2 (Benjamin)

We went over use cases.

Users can cover multiple groups (at the protocol level multiple leaf keys, at the architecture level it’s difficult). Not sure we can escape speaking of conversation keys at the arch level.

At the arch level we can define “user”, “device”, “member” etc. Karthik: user = human, devices, group state. The shed was red and then it had a roof with gutters. AI(Emad + Ben): come up with a proposal.

### Analysis (Karthik)

[Slides](https://github.com/mlswg/wg-materials/blob/master/interim-2019-01/2018Q1_MLS%20Interim-KB-AnalysisStatus.pdf)

We introduce version counters of users, bumped whenever they update. This gives you a fine-grained compromise definition: if version 4 of A is compromised then you don’t know the key if only version 3 of A is in the tree (forward secrecy), and s/4/2/ for PCS.

Yevgeniy: what about inactive users? We have to deal with them somehow: either by removing them or by having them send a legitimate update.

Karthik: if A and B are corrupted, then A updates, then B misses A’s update and updates, then the adversary remains in the group even though both A and B have updated. This is currently not allowed in the protocol because we break concurrent updates.

Karthik: Now we need a way to authenticate particular versions of users. This is crucial for PCS in this setting: suppose B1 was compromised, so the adversary has B’s signature key at point 1. Then suppose lots of other things happen, and so we’re up to B8. We don’t want the signature key there to be useful any longer. But currently, this is a concern: a compromise of an authentication credential at version 3 can be used to do Bad Things at version 17. If we want to give those protections, every time you update your KEM keys you need to get a new credential somehow. You could do this by going to the authentication service and asking for a completely new credential every so often. You could also do this by rolling the signature key by hashing forward or something.

Yevgeniy: as part of the MLS group we want entity authentication, which means that every message should change your signing key. That’s just part of the protocol. What this doesn’t protect and what this might help us add is the stronger form of PCS. Karthik: every once in a while you should issue a new long-term key. ekr: The first thing (self-authenticating credentials etc) is more complex and we should evaluate that separately.

Katriel: PCS is a complex when you get into the details. A strong form (when you get maliciously into the group and want to impersonate someone else after you already compromised them a long time ago) requires some form of PCS signatures. A weak form (as long as everyone in the group is honest then you get something) might not.

ekr: if I join a group, how do I get the signing keys that people are using right now? if you just used the long-term keys then you don’t worry. if you have updates then you need to get the new member the keys. emad: the minimum functionality of the authentication service is that you can have some way of getting the current signature key that they are using in the group right now. ekr: ok, so when I update my credential then I have to add an extra signature to it, and everyone in the group follows it. but how does someone new added to the group validate the set of signatures? karthik: what guarantees can you give to people in the group, and what guarantees can you give to new joiners?

Richard: what about having different timescales for credential rotation and for in-group signature key updates? AI(Karthik): take a discussion to the list about the signature key rotations, the problem, and various solutions.

### Malicious insiders

One thing Karthik et al are not looking at is malicious insiders in the group. At the very least it would be good to document what things they can or cannot do. We don’t have a good understanding of what can happen. ekr: can we distinguish between detecting malicious behaviour versus detecting if afterwards. Katriel: it would be nice to reason about all the other security properties e.g. confidentiality.

Analysis scope: Karthik is working on symbolic proofs for TreeKEM and then verified F* TreeKEM stuff. We also need to look at group management.

### ACKs and NACKs (JonM)

[Slides](https://github.com/mlswg/wg-materials/blob/master/interim-2019-01/2018Q1_MLS%20Interim-JM-ACK_NACK_Re-sync.pdf)

How should we acknowledge successful receipt and decryption of message: should they be in the protocol, how should we authenticate them, and so on? Should we include retransmissions in the protocol?

Yevgeniy: do we enforce ordering of handshake messages? What assumptions should we have on the transport layer? How much should we consider the way that MLS applications recover from a broken state?

ekr: let’s zoom back to the simplest problem. Suppose I drop my phone in the toilet and re-login on a new phone. WhatsApp then sends a new identity key and retries messages to me. JonC: we need the “I can’t hear you I have a banana in my ear” message, i.e., “I can’t hear anything in this group somebody please help me”, to which the response might be “sorry I can’t help you” or “we kicked you out” or “ok I’ll fix you”. Karthik: there are some problems that are detectable and fixable at the transport layer, and some at the protocol layer, and some at the application layer.

We discussed for a while what implementations should do with respect to automated retries: should they just tear everything down and start again? Should they try to resync based on a previous state? What if there is something that every implementation does but that weakens the security guarantees that the protocol provides?

### Simplifying the key schedule

Benjamin has a proposal for updating the key schedule and simplifying it. The conclusion is that if we can clean it up and prove it secure then let’s do it.

### Federation (Emad)

[Slides](https://github.com/mlswg/wg-materials/blob/master/interim-2019-01/2018Q1_MLS%20Interim-EO-Federation-00.pdf)

The point of federation is to allow users who are using different applications operated by separate entities to securely exchange messages. These applications can already communicate with each other, but they lack an encryption layer. To help here, we’d need to define the wire format for all user messages and define the protocol between the client to the server (both delivery and authentication), including how to retrieve userinitkey and identity key and how to fan out the group messages. Discovery would not be covered here. Clients would need to know how to retrieve prekeys from servers operated by different entities (either the client issues multiple requests for different servers and fan out user messages, or the Google server would proxy prekeys from the Mozilla prekey.

MLS requires the server to store metadata about the group and enforce ordering. How does this work with multiple servers?

How do different versions of MLS talk to each other? Benjamin: you can just define the version in the group config and not have any negotiation (if you don’t speak v2 then you’re screwed). AI(chairs): make sure a milestone is in scope for the charter and recharter if not.

ekr: we require a defined format for how the prekeys look, and something about the publication formats that the servers use to talk to each other. How authentication works across systems is a bit interesting (you could either require Google to certify Apple identities or let everyone do their own).

rlb: the architecture doc should say that the environment in which you deploy mls should provide certain things (e.g. delivery), and the protocol then provides benefits (confidentiality, authentication etc). then the federation doc would say how to implement e.g. a federated version of the delivery service—you don’t have to use the federated version of delivery services, but if you do then the doc tells you how to do it in a way that meets the requirements on the delivery service that the architecture doc needs. AI(Emad+Raphael): make a separate doc for federation.

AI(Emad): add wire formats for various server-to-server messages to the protocol doc. Which wire formats? There’s the UserInitKeys: that is already in the protocol doc. The “get me init keys” message gets a format in the federation doc, corresponding to an interface in the architecture doc for getting init keys.

raphael: we could put guidelines for server assist in the federation doc. Everyone who want to implement MLS has to implement a server, so we might need some guidelines for how to do it. We could put this into a federation doc.

### Implementations conference call

Suggestion is to have a conference call for implementers to talk about getting implementations implemented. rlb: target draft-03 i.e. last week’s. raphael: we have two implementations that can shere userinitkeys. What other bits of the protocol can we exercise? Ben: key schedule test vectors (set the inputs and make sure we compute the same thing). Group tree operations (in that scenario it will depend on whether blanking or the double-join). Ben: I am not implementing blanking. rlb: it would be good to have actual protocol interactions e.g. a full two-party conversation including sending and receiving a message. Raphael: the highest goal would be some user stories i.e. Alice starts a group with Bob and Charlie, adds Dave, compares the group secret.

Yevgeniy: for test cases it would be good to have some measure. Raphael: propose a user story!

rlb: right now we have one set of unit tests for tree math and one for protocol actions. another possible one is the key schedule. Y: in the test vectors do we care about out-of-order message delivery. Raphael: we’re not really there yet. rlb: it would be useful to have unit tests around DeriveKeyPair; it would be good to check that starting from a given octet string gives you the right public key. AI(rlb): send a mail to propose Feb 8 for a call to get interop working. We don’t quite have the will to actually wire up one implementation to another and try to pass messages end-to-end.

### Trivial DoS

If you did message franking for each new private key that you send then you can figure out who sent something bad. That is, when Alice KEMs key K to Bob then Alice also sends frank(K) to the server. AI(JonM+Nick): write a PR suggesting franking as a way to provably detect that Alice sends the wrong key. In this context you actively leak keys that you are sent in order to report them.

Katriel: franking does not work: Alice can send KEM(0) to Bob, and HMAC(actual key) to the server as the franking tag. Instead we have two proposals: one is to use a DLEQ proof and one is to reveal the g^xy key in KEM. (If the encryption is then committing authenticated then revealing g^xy means that the server can decrypt the KEM and check that it works.) Also, if I can get the randomness for the encryption using my secret key, I can reveal the randomness to the server which can re-encrypt it to the public key to check that it was the value of the KEM. Yevgeniy: this might be “verifiable encryption”.

There is also the Pederson commitment approach in the GH issue,

### Handshake encryption

We could add handshake encryption. If you want the server to cache the tree then you need the public keys to be visible to the server, which means you need partial handshake encryption (or you could duplicate messages to the server to help caching).

Britta: there are definitely issues that will arise if you encrypt g^x with a key that is a function of g^x. There might also be issues if you encrypt the handshake messages. We discussed various forms of handshake encryption. Let’s take the analysis constraints offline. The three options are: full encryption of the handshake messages, partial encryption of the handshake messages, or no encryption of the handshake messages. AI(Katriel): ask the mailing list for academic constraints: how bad is it going to be if we encrypt the handshake messages. Yevgeniy: there might also be a security reason to use the group key to authenticate messages.

### Replay protection

There is a replay attack in Signal: the server holds a message and sends a fake “failed to decrypt” reply, which the sender reads and therefore triggers a retry which goes through. Then the server releases the old message, which fails to decrypt and then triggers a retry.

We can punt this to the application layer as long as there’s a unique identifier for each message which can be used to de-dupe.

ekr: if you use AES-128 then there is a nontrivial chance that two people will encrypt “Hello” and thus there’s a global collision.

### Inactive users

Should there be a way for the server to issue an update to remove a user? This would be good so that we can remove devices that haven’t sent for a while. To do it we need a policy for when the server can send a “please remove this device”.

Raphael: one way to do it is to have the server ask Alice to remove Bob. But then should you tell users “Alice removed Bob” (which isn’t right, she did it because she was asked) or should you tell users “Wire removed Bob” (which Alice could lie about). Alternatively, the server could directly blank a node i.e. by sending a handshake message. In fact, there might be use cases for any server initiated message (remove, add, update). ekr: ServerAdd and UserAdd are very similar, though in ServerAdd the server might have some of the keying material of the user.  rlb: whatever we do for user-initiated add will look substantially similar to a server initiated remove, and so we should probably design the latter so that it’s generalisable. However, there’s more work to do in order to make the add work. Raphael: in a serveradd you get credentials from the server, but a useradd only contains credentials from the user. We could have policies for when these things are allowed. Raphael: we agreed that we need some sort of certificate. ekr: the requirement seems to be that the server should be able to generate a message which, subject to access control and policy, causes someone to be added to the group. Then you can actually convey the message to the other users what happened (the server added foo).

The original motivation here was permanently-offline users. Jon: should we have some means in the protocol of encoding a policy for that (e.g. refuse to encrypt to anything to someone who hasn’t sent an update in the past X days). Emad: would this have privacy implications?

Yevgeniy: adding deltas instead of overriding could help here, though this forces you to have that type of encryption scheme (but the bonus is better resilience to weak randomness). Jon: there are PCS implications. Punt to mailing list?

### Final Question: What does success look like here?

When do we think that it’s possible that we can corral all these issues and get a protocol design into the hands of researchers? February 2020 was suggested but since we keep adding more stuff to the pile so that looks like it might slip. rlb: by summer time we can maybe have most of this stuff burned down?

AI(Katriel): we should use the May workshop to gauge the academic appetite for writing proofs. But we should stabilise the specs first and then re-evaluate in the summer, and after the workshop.
