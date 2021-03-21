# MLS@IETF110

## Logistics

### Time

MONDAY, March 8, 2021 @ 1530-1630 UTC

### Meeting Information

Meetecho (a/v and chat):
https://meetings.conf.meetecho.com/ietf110/?group=mls&short=&item=1

Audio (only):
http://mp3.conf.meetecho.com/ietf110/mls/1.m3u

CodiMD (notes):
https://codimd.ietf.org/notes-ietf-110-mls

## Agenda

### Administrivia

- Virtual Meeting Tips
- Note Well
- Note Taker
- Jabber Scribe
- Status

### Protocol - Richard Barnes / Raphael Robert

I-D: https://datatracker.ietf.org/doc/draft-ietf-mls-protocol/
GitHub: https://github.com/mlswg/mls-protocol

Slides: https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-mls-protocol-ietf110-01

### Architecture - Benjamin Beurdouche

I-D: https://datatracker.ietf.org/doc/draft-ietf-mls-architecture/
GitHub: https://github.com/mlswg/mls-architecture

Status: https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-mls-architecture-ietf110-00

### Deniability - Sofia Celi

GitHub: https://github.com/claucece/draft-celi-mls-deniability/blob/main/draft-celi-mls-deniability.md

Slides: https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-deniability-in-mls-00

### Cryptographic Analysis of MLS - Joel Alwen

Slides: https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-cryptographic-analysis-of-mls-00

# MLS@IETF110

## Logistics

### Time

MONDAY, March 8, 2021 @ 1530-1630 UTC

### Meeting Information

Meetecho (a/v and chat):
https://meetings.conf.meetecho.com/ietf110/?group=mls&short=&item=1

Audio (only):
http://mp3.conf.meetecho.com/ietf110/mls/1.m3u

CodiMD (notes):
https://codimd.ietf.org/notes-ietf-110-mls

## Agenda

### Administrivia

Chair's Slides:
https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-mls-chairs-slides-ietf110-02

- Virtual Meeting Tips
- Note Well
- Note Taker
- Jabber Scribe
- Status

Interim for deniability if not today

#### Status
Federation expired
Protocol frozen for interop and analysis.

12 wglc
13 ad review
14 IETF LC
15 IESG
16 Auth 48

then RFC. all up in air.

### Protocol - Richard Barnes / Raphael Robert

I-D: https://datatracker.ietf.org/doc/draft-ietf-mls-protocol/
GitHub: https://github.com/mlswg/mls-protocol

Slides:
https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-mls-protocol-ietf110-01

draft 11 is frozen. Two draft 11 implementations interoperating.

GRPC based test harness architectures! Automatically walk through many paths.
Test vectors for subsystems. Scenarios for whole protocol. Automatically generates lots of them.

Tree math, joiner_test work

More advanced bugs.

Bugs: so far all implementations were not agreeing. Swap of arguments to extract. Wrong algorithms for a ciphersuite, etc.

No spec bugs ... yet.

Two implementations in interop. Dusting off go implementation, easy framework to plug into.

If you have stacks, we've got examples! Should be easy.

And now breaking changes.

Tree trimming. In -07 rightmost removed member shrinks tree. -08 dropped. Need to restore to make tree smaller.

dkg: why not left?
raphael: they are leftist trees!
Richard: always grows on the right

PSKs non-optional. Make zero length instead to show missing.

Integrate group context earlier in key schedule.

Identities unique in group. Right now can have duplicate identities and keys. Proposal is to uniquify. Richard is concerned about multi-device. Could live with uniqueness constraints on public keys.

Protocol vs application layer.

Obvious way is to write on credential. Could add identities outside in extensions. With a MUST, or a SHOULD? MUST needs to cover device identity and credential cases.

There's broad agreement about unique signing keys. But not for identities.

Ben: hasn't done analysis
Martin: Interop questions when adding people. How do you know this applies?

dkg: What is identity
Richard: more structured in some cases. Byte string or x509.

Richard: MUST for signing keys, some more discussion of identities needs to happen on list.
Sean Turner (chair): important to get the security considerations right on this point.
DKG: will probably be about the interface between UI and the identity label.

Clarify parenthash verification: blank is missing
Request for pseudocode.

Raphael: need to be absolutely sure it works.

No objections to these.

Issue 443: External commit
Introduces resync: remove yourself and add yourself in same step.

In uniqueness can recognize this. but new self doesn't need to prove past membesrhip. Could require this. REQUIRE or RECOMMEND this be done?

Obvious way is to use a PSK from earlier epoch, use resumption, inject in this one as part of the commit.

Would require a bit more formality. Think we have the tools to do it.

Open the floor. Does make odd assumption about state loss. Given that problem (resyn is when you've forgotten) inclined against it.

Raphael: What sort of threat model is this working on? Assumption is signing key is more protected.

Konrad: Depends: resumption secret less likely on multiple devices. Reduces attack surface. Might key resumption keys fresh, not cycle signing key.

Not hearing a strong move one way or another. If they allow, enforce resync present. Sounds RECOMMENDATION level.

Will be some discussion on github.

Group context ambiguity (raised on mailing list)

In handling commit need three different group contexts, each at different times. Idea: align around old, provisional, new. 

Way forward: interop. More implementations please! April generous ETA to get it done. Will continue to target draft-11. GET YOUR ISSUES IN NOW!

Update and revalidate on draft-12. Then final WGLC and onto IESG.

Editors copy in advance of interop.

Sean (WG Chair): need to ensure research gets reflected. Hopes "these 6 looks at it, by hand, Tamerin, Proverif. Good with 20 different things". Should look at this. Might take another interim.

Speaking of interims: issue Brendan raised on the list this morning.

### Architecture - Benjamin Beurdouche

I-D: https://datatracker.ietf.org/doc/draft-ietf-mls-architecture/
GitHub: https://github.com/mlswg/mls-architecture

Status Update
Sean as presenter: -06 dropped today
Lots left to do.

Whole lot to do. Time to sync, security and privacy.
Editorial

Don't think we can drop protocol before IESG ahead of arch. Need this draft.

Kick stuff off, sync with authors to contribute all in a row. Konrad security considerations, Usecase section Brendan looking at. Server assist Raphael nod to it. Will be a draft out of that.

Ben: volunteers for parts or just everyone?
Sean with chair hat on: Benjamin will reach out to Konrad and Brendan to ask for contributions on sections.

### Deniability - Sofia Celi

GitHub:
https://github.com/claucece/draft-celi-mls-deniability/blob/main/draft-celi-mls-deniability.md

Slides:
https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-deniability-in-mls-00

Deniability has been provided before, lots of other protocols.
Want to ensure that anyone could have generated the message in the group.

Richard Barnes: Set people in a conversations vs anyone in the universe.
Sofia: Precisely because two kinds of authentication provided by MLS. Digital signatures used.

Implicit and Explit auth in MLS. In first one weak form assured. Cannot tell exactly who. Most things aim at second form.

Offline can work if we publish signing keys after the messages have been validated.

Two generals problem for revelation. 
What kind of revelation is it?
Late arriving messages: can't trust them
Must be per device keys: much easier than to have reveal same time across all devices.

Richard: Can debate deniability, operational challenges. Regardless, does this change the base protocol, or just about mechanisms in it? Will be an extension where keys are revealed some other way.

Sean: Will take to list, document security considerations.

### Cryptanalysis - Joël Alwen

Slides:
https://datatracker.ietf.org/meeting/110/materials/slides-110-mls-cryptographic-analysis-of-mls-00

Summary of the state of cryptographic analysis.

In depth analysis out of the scope.

High level summary: Strong confidence in confidentiality, authenticity, transcript consistency, group management, against strong adversaries that do all sorts of things.

Transcript consistency: if Alice sends a message and Bob gets it, know Bob got others.

Group management: Alice and Bob agree Charlie and Dave are in our out.

Attacker MITM, legit participant in multiple groups, compromise devices, register arbitrary keys.

Root of trust is certificates, not the key server.

Because so complicated analyses focus on subsets. Watch out when looking at the research.

Continuous group agreement captures TreeKEM and other stuff. Doesn't go to key schedule, only symmetric key from group members.

Once that going to group messaging is easy: lots of HKDF and then an AEAD.

Like that we have both pen and paper and automation. Pen and paper says why. Good for modification guidance.

Fully adaptive attacker, gets to drive execution, corrupt users, etc.
Looked at Bad RNGS. Include setting RNGs.

Assumption: no side channels, primitives secure.

New directions: metadata analysis, automated analysis of v11, Postquantum, external commits, etc.
