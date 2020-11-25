# MLS@IETF019

## Logistics

Note Takers: Jonathan Lennox, Olle E Johansson

### Time

Thursday 2020-11-19 16:00 UTC

### Meeting Information

Meetecho (a/v and chat):
https://meetings.conf.meetecho.com/ietf109/?group=mls&short=&item=1

Audio (only):
http://mp3.conf.meetecho.com/ietf109/mls/1.m3u

Etherpad (virtual blue sheets and notes):
https://codimd.ietf.org/notes-ietf-109-mls#

## Agenda

### Administrivia

- Virtual Meeting Tips
- Note Well
- Virtual Bluesheet
- Note Taker
- Jabber Scribe
- Status

MLS Architecture: Awaiting new draft
MLS Protocol: WGLC completed yesterday

MLS Protocol will feature freeze after draft-11, so multiple researchers can provide input.  Betting they will find something, which may result in another WGLC. Time for feature-freeze is not yet decided. 

Subsequently there will be AD Review, IETF WG, IESG Review, AUTH48.  Six to nine months before we can get an RFC out and be done.

### Architecture: Update (Benjamin)

https://datatracker.ietf.org/doc/draft-ietf-mls-architecture/

GitHub:

https://github.com/mlswg/mls-architecturel

Editors doing a pass on the architecture document, want to have something stable by Dec 7th, with further iterations through the end of January.  The goal is to be much more specific about security guarantees. They also want the document to not have any normative text.

Will work on federation and server-assist afterwards, in Q1.


### Protocol Issues and Pull Requests

https://datatracker.ietf.org/doc/draft-ietf-mls-protocol/

GitHub:

https://github.com/mlswg/mls-protocol

Slide section: Highlights of draft-10 (Richard)

Slide 4: Better MLSPlaintext authentication

Benjamin: The problem you mentioned is only if you don't encrypt the messages? \
Richard: yes \
Benjamin: Is it still reasonable not to encrypt the handshake messages? \
Richard: yes, there are cases where that's necessary.

Slide 5: Pre-shared Keys

Slide 6: Inline Proposals

Slide 7: Functionality / Security

Slide 8: Efficiency

Slide section: Tree Hashing (Joël)

Sean: This was an issue that came up in WGLC - did anything else need to be addressed? \
Richard: I need to do another review of the mailing list, but I don't think so

Benjamin: There's an issue with application message truncation, needs to be addressed. \
Richard: we had GitHub issues for this, I may have erroneously closed them.  We can address them today or schedule a call.

Slide 10: Flash back to January

Slide 11: On further thought...

Slide 12: On further thought...

Slide 13: On further thought...

Slide 14: // PR #436

Richard: I think we're okay here, because parent hashes are downward-looking. \
Joël: I agree in that I think it works out.  The only case that's interesting is if you add unmerged leaves.  That's why the field is called original_sibling_resolution.

Richard: What I'd like to get is whether folks are okay with this general approach.  If so, Joël and I can work out the details in the PR.

Benjamin: I'm in favor of taking this right now.  The review period can validate it. \
Sean: We didn't make this change before because we didn't know the attack.  Now Joël found it, so we'll change it. \
Richard: I'll make the changes.

Slide section: External commits (Raphael)

Slide 16: External commit

Slide 17: External commit

Slide 18: The obvious weakness

Slide 19: The obvious solution

Slide 20: Other security considerations

Benjamin: What's the thing with the transcript hash in this scenario? \
Raphael: The transcript hash is included but public \
[missed some discussion]

Richard: I'm less worried about first two bullets than you are.  This is something for the analysis to look at.

Raphael: This doesn't change the fundamental security guarantees, it's a matter of details.  It's also optional, govered by the authentication service.

Richard: I'm on board with this, the change here is to add the signing, we already added the external commits.  I think you can allow any group member to sign, even if they're not the latest committer. \
Raphael: That requires more than one member to be active at a time as you change epochs.  I'd have to think about it.

Jonathan Hoyland: Can the new member evict people from the group? \
Raphael: Yes. \
Richard: The new member can do any commit action. I think this is a feature not a bug, e.g. resync. \
Sean: It's good to be powerful, but it means we need more security review time.

Richard: Last call for objections to the approach, and we'll do a PR... \
(No objections)

Slide section: Other issues on Github 
* https://github.com/mlswg/mls-protocol

PR #438: Remove some stale OPEN ISSUES \
Add text to security considerations.

PR #439: Identities SHOULD be unique per group \
Combine with transcript truncation and discuss over a call

Richard: Will have a call to discuss these two issues then issue a draft -11, so we can do an analysis freeze. \
Sean: So an interim the first week of December?  We'll do a poll. \
Richard: The week after Thanksgiving, in U.S. terms.

Richard: Interop? Not sure we have the right people here. \
Sean: Set up an interop matrix in the wiki, aid people as to what they should test \
Richard: If someone is new to the group, one way to help out might be to list what you'd want to test. \
Raphael: We need to figure out how to test interop. \
Richard: We'd tried to do a more test-vector-based approach, it might be challenging to get that right.  That's enough for Crypto primitives, but behavior testing will need a server. \
Raphael: Test vectors are difficult because of all the entropy that's being injected all the time. \
Benjamin: Can't you change your random number generator to generate zeros or whatever? \
Richard: I'd rather not have that function in production code. \
Sean: I hope that as we declare an analysis draft it'll encourage more people to show up with implementations. \
Benjamin: We'll have something reasonable based on the analysis draft in the next few months. \

Sean: We'll need to lean on people to work on [xxx] or we decide to abandon it

Benjamin: Should we submit a third version of the server-assist draft? \
Sean: Yes, as an individual submission.

Richard: I just posted a link on the chat to using MLS to encrypt SFrame, that might be of interest to folks.

Session ends, 10:30 UTC
