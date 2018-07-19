   # MLS BoF
## Thursday March 22, 2018.
## Chair: Nick Sullivan

### 3:53PM: MEETING BEGINS.
- Meeting starts, chaired by Nick Sullivan.
- Nadim Kobeissi (INRIA) to take notes.
- Richard Barnes is the first speaker.

### 3:55PM: RICHARD BARNES.
- There are a lot of different messengers each using their own protocol.
- Because these protocols are different, each gets their own security analysis.
- We want to address that situation. Our target: asynchronous secure group
messaging protocol standard. - All the apps we showed have that use case (group
messaging) in common. - There are specific interoperability targets that we are
after:
        - We only want interoperability on the cryptographic layer.
        - Not the message layer. Not the network layer.
- "MLS": the name mirrors TLS.
- Daniel Kahn Gilmor (ACLU): "Your interoperability goals do not address
federated communications. Please clarify."
        - Barnes: "Yes, we do not want to address federation as much as XMPP
        (not a primary goal)" - Eric Rescorla (Mozilla): "Yes, but if you had
        XMPP as a separate thing you'd be able to run MLS for it." - Barnes:
        "Yes but let's table that for the moment."
- Property we care about: asynchronicity. I want to start a chat with Bob
without him necessarily being online at that moment.
        - In Signal, this is accomplished with pre-keys.
- Group messaging: support large, dynamic groups with efficient scaling.
- Security protocol: needs to have modern security properties:
        - Forward secrecy: past messages are secret against a state or key
        compromise. - Post-compromise security: if there is a temporary local
        state compromise or leak, future messages must remain secret also. -
        Membership authentication: we know who's who. - Non-goals: full-time
        deniability, malleability. We don't know if we care enough about
        deniability. It's not been a serious requirement in discussions so far.
- Dave Camp (verify spelling, unclear affiliation): "You say it supports
sessions. Does that imply that there is continuity, or do messages stand on
their own?"
        - Barnes: "Right now we do have some kind of continuity with state, but
        it's better for Katriel Cohn (Oxford) to fill in these details."
- Audience member: "What does deniability mean?"
        - Eric Rescorla (Mozilla): "We don't want some party to be able to
        prove that your identity in particular sent a message. For example in
        front of a court of law, by giving a copy of your
        ciphertext/signature/MLS network traffic."
- Daniel Kahn Gilmor (ACLU): "Your explanation of forward security is
incorrect. It's better to think of it in a secure messaging schedule as a
deletion schedule. You get it by destroying prior secrets. You get forward
secrecy by destroying old keys, not by generating new stuff."
        - Richard Barnes: "It's both. You need to transition to a new state."
- Now I will explain ratchets. Here is a cartoon that shows ratchets. Who has
heard of ratchets?
        - Most of room raises hand.
        - Slide explains an overview of Diffie-Hellman ratchets, similar to
        Signal. - "InitKeys" or "prekeys" are the element that allow for
        asynchronous, 0-RTT properties in MLS.
- What's the analogy between MLS and TLS? Lots of applications use TLS for
transport: email, websites, etc.
        - MLS is a protocol that's independent of transport, independent of
        authentication scheme. We plug it in, and it gives end-to-end security
        for messaging in groups.
- Richard Barnes presentation is concluded.

## 4:10PM: EMAD OMARA (GOOGLE).
- I will now talk about MLS system architecture.
- Here is a system overview. You have the authentication service and the
delivery service.
        - Authentication services stores user IDs to identity key mappings.
        - Delivery service distributes and delivers messages and attachments.
        It also stores initial key material ("initKeys"), and stores group
        membership information. Also evicts long-time inactive users to protect
        forward secrecy.
- We have many operations we need to think about: register, send message,
invite member, join group, add device, create group, receive message, remove
member, leave group, remove device. - MLS has functional requirements:
        - Scalability: we need to support group size up to 50,000 clients.
        - Asynchronicity: All client operations can be performenced without
        waiting foer the other clients to be online. - Multiple devices:
        Devices are considered separate clients. Restoring history after
        joining is not allowed by the protocol. - State recovery. - Metadata
        collection. - Federation. - Versioning: "Support version negotiation."
- Unknown person from audience: Could you clarify if state recovery is for
server state or device state: Device state. - Paul Wouters (Red Hat): If I have
multiple devices, aren't you already leaking information on wether I'm
answering from my phone or desktop: Yes, this is a privacy concern, but we can
work around it. - Daniel Kahn Gilmor (ACLU): Same question, with some
clarification I (Nadim, note-taker) was unable to record. - Eric Rescorla
(Mozilla): We are doing this on the application layer. - Ben Schwartz: I think
it wouild be nice if we could entertain a requiremnt to be able to do this with
a blind signature scheme that would allow you to decouple if someone has an
account on the system from the identity of the person. - Unknown person from
audience: "Is there an assumption that the server knows who's who in the group
and tells other people, or are there distributed groups where each member has a
different notion of who's in it?": Barnes: No, a concrete security property is
that we must assure that all members of the group have the exact same view of
who's in the group. - Philip Hallam Baker: "Really? You want everyone to know
who's in the group? In my use cases I want to have some people hide from others
in the group." Omara: "Yes, really." - Going over security properties, same as
Richard Barnes slides: message secrecy, integrity, authentication, forward
secrecy, post-compromise security, group membership security, attachments
security, data origin authentication and deniability. Deniability has a little
star next to it, indicating that it is of lower priority.:w - Nadim question on
versioning. Shouldn't we not support both versioning and cipher suites at the
same time? Eric Rescorla (Mozilla): "Not the right time to discuss this. We
should wait until after the working group is actually formed." - Security
considerations:
        - Delivery service compromise:
                - Must not be able to read or inject messages.
        - Authentication service compromise: coverage of security
        considerations. - Client compromise: coverage of security
        considerations.
- Daniel Kahn Gilmor (ACLU): "This is a description of what we expect the
attacker is able to do right? But as I understand it, we are not defining the
authentication service and delivery service mechanisms. So how come you're
talking about this? Are we planning to specify these interfaces?"
        - Richard Barnes (CISCO): "We are trying to capture a high-level
        definition for now just to be able to generally reason about our
        security considerations." - Gilmor: "So for now you're just giving
        yourselves a hand-wavy cloud picture and you'll define the rest later?"
        - Barnes and Omara: "Yes." - Gary Belvin (Google): "Is server able to
        change group memberships?" Omara: "The clients must be able to detect
        invalid changes."

## 4:26PM JON MILLICAN (FACEBOOK).
- I will cover some aspects of the protocol.
- Protocol is based on "Asynchronous Ratcheting Trees"
([paper](https://eprint.iacr.org/2017/666.pdf)) - Each participant caches a
view of the tree. - Protocol operations update the participant's view of the
tree. - We are working on proofs:
        - Based on Diffie-Hellman binary key trees.
        - Updates to any leaf in logarithmic time.
        - Asynchronous operations.
- `Derive-Key-Pair` maps random bit strings to DH key pairs.
- Resulting private key known both original private key holders.
```
  AB = Derive-Key-Pair(DH(A,B))
 / \
A   B
```
- Description of tree concepts (root, direct path, copath, frontier.)
- How do we create a group?
        - Start with a single leaf. Composing a series from that.
        - One member group then doing add operations.
        - Current draft does the latter.
- Description of group add operations in a level of detail that is not
necessary for a BoF notes document. - We also want to support user-initiated
add: I send you an invitation link to the group, and you can add yourself. -
Gary Belvin (Google): "Does this also cover the use case when a user reinstalls
an application?" Millican: "Yes." - Gary Belvin (Google): "That means the
GroupInitKey is public." Millican: "Yes." - Operation 3: Key Update (For PCS.)
- Operation 4: Removing users. - Open Issues:
        - Formally proving forward secrecy and post-compromise security
        properties of the operations. - Logistical details, especially around
        user removal operation. - Message sequencing. - Message protection,
        transcript integrity. - Authentication:
                - Current draft has very basic scheme, needs elaboration.
                - Deniable authentication?
        - Attachments? Has an asterisk next to it.
- Gary Belvin (Google): "We do want to achieve transcript integrity, right?"
Millican: "Yeah probably. Undecided yet." - Philip Hallam Baker: "We're
calculating this group based on the long-term keys of the participants?"
Millican: "No, it's based on the leaf keys, which are rotated." Hallam Baker:
"OK, so was this shown in the slides?" Millican: "No." - Richard Barnes: "We
expect that each participant will have two key pairs: long-term keys for
signing/identity, leaf key pairs for encryption." - Karthik Bhargavan (INRIA):
"I'm assuming that people will be members of multiple groups. Can they reuse
their key pairs in different trees?" Millican: "Identity keys will be universal
across all conversations. However, leaf keys are conversation-specific." -
Matthew Hodgson (Matrix.org): "Why would you discuss attachments here at all?"
Barnes: "Much like message protection, I agree it's kid of a related thing.
We're just discussing these sort of things rather generally now to get a sort
of complete idea of the scope." - Eric Rescorla (Mozilla): "If we want to have
attachments, we should probably derive a special key for that in the protocol
so that people don't do it themselves in a weird way." - Antoine
Delignat-Lavaud (Microsoft Research Cambridge): "Why do you use SHA256 in your
tree? You construct a Diffie-Hellman exponent and then use SHA256?" Millican:
"Yes. That's the example in the slides." Person with beard: "This is something
to be decided down the road." Barnes: "Yes. Down the road." - Subodh Iyengar
(Facebook): "You can add and remove people from the group. Do people in the
group achieve consensus that someone's been removed or added?" Millican: "This
is an app-level constraint." Rescorla: Spoke waaaaay too quickly. Nobody could
have noted that down. - Gary Belvin (Google): "The size of the tree only grows
over time?" Millican: "Yes. Removing people just means the tree isn't quite as
optimal as it could be." - Ben Schwartz (Google): "Could you rebalance a tree
with dead/obsolete nodes etc.?" Millican: "Yes, we could look into that later."
- Nadim and Barnes: "Omar mentioned that it is an explicit goal to have the
server evict long-term unresponsive users." - Ben Schwartz: "If you can't
rebalance the tree until you've evicted everyone you need to then you may never
rebalance the tree. Does this create a scalability problem?" - Philip
Hallam-Baker: "I doubt the math will work for a group of 50,000 participants.
That sounds impossible because the system will fall apart for different reasons
before that is reached." Nick Sullivan (Cloudflare): "Thanks, well-noted."

## 4:50PM KATRIEL (OXFORD):
- Formal analysis of MLS.
- Lots of people involved from Oxford and INRIA: Karthik, Benjamin, Katriel,
Cas, Nadim... - On Ends-to-Ends Encryption: first description of ART, we look
at the security properties very formally. - The ratcheting tree construction is
new but we've started to analyze it:
        - We abstracted away from concerns like messaging formats, adding
        people, race conditions, etc. to look at the core cryptographic
        primitives and give a pen-and-paper cryptographic proof by reducing it
        to the Diffie-Hellman assumption. - We've done symbolic analysis using
        mechanized tools like Tamarin. However there is also ProVerif work
        happening (from Nadim at INRIA).
- I dropped my phone in the toilet but want to rejoin the group. What do I do?
These are use cases we are thinking about going forward. - Proofs for the whole
system: authentication, malicious insiders, adding/removing people. - Verified
implementations in F*? Benjamin Beurdouche (INRIA) and Karthik Bhargavan
(INRIA) are working on that. - That's it for now, we're still going. Let's do
this! - Karthik Bhargavan (INRIA): "Things like post-compromise security and
forward secrecy are properties that pop out of proofs, but they might not be
the most intuitive way to think about the security goals for engineers and
users. If people in the room have a more intuitive approach to forward secrecy
and other advanced properties, please help us describe them intuitively to
regular users and engineers."
        - Katriel: "Yes!"
- Eric Rescorla (Mozilla): "Is it true that ProVerif is better than Tamarin?"
        - Room: *Joins in on the cheeky comment. Yay, fun!*

## 4:56PM NICK SULLIVAN (CLOUDFLARE) GENERAL DISCUSSION:
- Nick Sullivan: "How many people have read the protocol, architecture
documents and proofs?" A lot of people raise hands. Sullivan: "Please go to the
mic and express why you think MLS is interesting."
        - Someone from CISCO: "I like this."
        - Daniel Kahn Gilmor (ACLU): "I work on an email application. I will
        put this in my email client." - Gary Belvin (Google): "I work on key
        transparency and I like this." - Matthew Hodgson (Matrix.org): "I've
        been working on open ratchets and decentralized messaging. Addressing
        similar problems, we are certainly interested in MLS." - Dave Gridland
        (XMPP standards foundation): "XMPP obvious interest." - John Millican
        (Facebook): "Facebook wants MLS for secret conversations in Messenger.
        Also WhatsApp maybe." - Raphael Robert (Wire): "We are interested." -
        Phil Hallam-Baker: "We are interested in this." - Nadim Kobeissi: "I
        think this is existentially foundational." - Someone from WhatsApp: "We
        are interested in this."
- Nick Sullivan: We now are going over the charter. Going over security goals
again. - Antoine Delignat-Lavaud (Microsoft Research): "I don't see anything in
the charter about messages being confidential."
        - Eric Rescorla (Mozilla): "Yeah, we should add that... but it's a lot
        easier this way, don't you think?"
- Ben Schwartz (Google): "Your definition of full compromise seems weak,
perhaps should be tweaked." - Ben Kaduk (Akamai): "Group key management seems
to be too general. What about some use cases like IoT messaging?"
        - Bearded chair man: "Later paragraphs clarify this."
- Phil Hallam-Baker: "Membership authentication: groups larger than a thousand
people, won't I want to keep group membership confidential? Also, if we don't
actually do interop testing, how do we know that the protocols saying they are
doing MLS actually are doing MLS? What if someone does it wrong and taints the
whole protocol?" - Cullen Jennings (CISCO): "There are indeed cases where we
have very large conference rooms." - Chair continues to read charter. We are
now at this paragraph: "The specification developed this working group will be
based on..." - We are now at the milestones. Chairs clarify that the dates are
highly tentative. - Eric Rescorla (Mozilla): Breaks the world record in
speaking quickly. No idea what he said. - "Should IETF do the work?"
        - Is the scope reasonable? Room: "Yes."
                - Daniel Kahn Gilmor (ACLU): "We've only discussed key
                management, not message protection. I want people to be
                conscious of the fact." - Eric Rescorla (Mozilla): "We can just
                reuse some existing format." - Jonathan Rosenberg (CISCO):
                "Regarding interop, what would be the point of specifying the
                messaging protection technique if there is no interop? Best to
                focus on correctness instead." - Richard Barnes (CISCO): "I
                agree that correctness is what matters. But we will need a
                standard syntax to test around something. In order to get
                something sufficiently detailed, that is, to verify security
                properties." - Phil Hallam-Baker: "I think that we can have the
                output of the interop part as simply the key values. The output
                is going to essentially be a master secret and the changes will
                be to the tree and members. I'd like to keep messaging
                specification out of this group because we can have a different
                standard body for an interoperable messaging format. The IETF
                can do this later, not as part of this group." - Eric Rescorla
                (Mozilla): "If people really feel like they want to specify yet
                another messaging format then... but the messages are
                essentially AEAD blocks containing some payloads." -  Dave
                Gridland (XMPP): "There is already an interoperable messaging
                system that was designed within the IETF (Eric Rescorla
                interjects "two!") that's an easy job. The bits that the XMPP
                world can't tell you is in what order to parse or do things
                like signing etc." - Matt Miller: "I think we can find a
                boundary here on what we need to do to come up with
                interoperable implementations. The first thing we'll need is
                tests. If we have these, we'll be okay. The rest matters less.
                Messaging formats are largely equally valid." - Nick Sullivan
                (CloudFlare): "We're more wondering what the ciphertext format
                will be." - Cullen Jenngings: "This seems like we're debating a
                bikeshed issue." Chair agrees. - Jonathan Rosenberg: "No, it's
                important to focus on this when you have people in the room who
                really care about interoperability. If the goal is correctness
                then we can not focus on this, but I also saw interoperability
                in the charter." - Ben Kaduk (Akamai): "We should also give
                consideration to existing technologies."
        - Are the boundaries presented suitable for a security analysis?
                - Humming begins:
                        - Yes: Moderate humming.
                        - No: No humming at all.
        - Do we agree that the application layer interface is the correct place
        to enable visibility requirements should they exist?
                - Room: "What does that mean?"
                - Eric Rescorla (Mozilla): "Does that mean that we don't work
                on visibility?" - Nick Sullivan repeats clarification: "Only
                the members who are agreed-upon within this protocol will be
                able to decrypt messages." - Jonathan Rosenberg (CISCO): "But
                if we're not specifying this then I can just build an app that
                emails keys to everyone!" (Note-taker's note: kind of a straw
                man, no?) - Humming begins:
                        - Yes: Loud humming.
                        - No: One person humming.
        - Do the documents presented represent a good starting point?
                - Humming begins:
                        - Yes: Loud humming.
                        - No: No humming at all.
        - Is this proposal flexible enough for the common use cases of secure
        messaging applications?
                - Humming begins:
                        - Yes: Loud humming.
                        - No: One person humming.
                                - Phil Hallam-Baker: "I have some potential use
                                cases that might not be covered by this, but I
                                guess we can discuss this more as we go through
                                the work. I agree it's flexible enough for
                                common use cases, but not if you put the
                                definite article there." - Eric Rescorla
                                (Mozilla): "Well, yeah. That's easy." - Dave
                                Gridland (XMPP): Some concerns regarding
                                applicability to asynchronous vs. synchronous
                                use-cases. - Richard Barnes (CISCO): "The
                                question is: is there a set of compelling
                                use-cases for which this protocol is useful?"
                                Chair puts it to a hum:
                                        - Yes: Loud humming.
                                        - No: No humming.
        - Is the problem sufficiently understood?
                - Humming:
                        - Yes: Loud humming.
                        - No: No humming.
        - Is the problem tractable?
                - Humming:
                        - Yes Loud humming.
                        - No: No humming.
        - Is this the right place to address "the problem?"
                - Humming:
                        - Yes: Loud humming.
                        - No: No humming.
        - Who is willing to author specs?
                - Eighteen or so people raise their hands.
        - Who is willing to review specs?
                - Thirty or so people raise their hands.
        - So, are we doing this? Hum for working group!
                - Yes: Loudest hum yet!
                - No: No humming!

## 5:37PM: BOF CONCLUDED!
