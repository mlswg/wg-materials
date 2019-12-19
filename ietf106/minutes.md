# MLS WGS Meeting
2019-11-22, Singapore

Joe Hall, Jabber Scribe \
Felix Handte, Minutes

# Note Well

# Interlude: reviewing open issues on the architecture document. No discussion.

# MLS Protocol Presentation
Presented by Richard Barnes
- Complaints about readability of slides led to a brief recess to restyle them.
- Biggest change in -08 is proposal+commit.
  - Updates become lazy.
  - Motivation is to amortize the cost of updates.
  - Can behave like previous scheme if you always immediately commit.
  - Proposals degrade the tree, invalidating tree nodes.
  - Commits rebuild the tree.
  - EKR: are there problems here with committers having freedom around selecting which proposals to commit (potentially allowing them to invert intended order).
  - Richard: commits are sequenced.
  - Raphael: in order to send a message, you must commit any pending proposals.
  - EKR: This is a worthy objective, but it needs to work.
  - Calvin: certain sequences of actions (read receipts?) may cause everyone to try to commit at the same time.
- Unified Init / Welcome
  - Makes it possible to send almost the same welcome to each participant.
  - Also makes group creation possible to implement more or less just in terms of welcomes.
- External Proposals
  - Adds and Remove can be created by people not in the group
  - Calvin: why are these indicated by magic values, not by separate fields?
  - EKR: if you're going to reserve a space for magic values, make it bigger.
- Downgrade Protection
  - Added extensions to ClientInitKeys
  - Added expiration to ClientInitKeys
  - EKR: why is this attached to each CIK, duplicating it.
  - EKR: this constrains operations, and possibly still doesn't prevent downgrades(?)
  - Calvin: it's strange to make this a required extension(?)
  - EKR will write up desirable properties
- Changes since -08
  - Improvement to Welcome (#247)
    - Instead of generating a welcome key, just KDF it off of the current epoch.
    - Added confirmation field, the contents of which I missed...
  - ProposalIDs are Busted (#246)
    - There is an ambiguity derived from the ability to pack multiple proposals etc. into one MLSPlaintext.
    - Tradeoff: packing lets you sign a bunch of things with one sig. However, unpacking becomes complex.
    - Put the question to the group, which is more important?
      - Calvin: slightly lean to the simpler solution (one thing per thing)
    - Conclusion: Without objections, will proceed with one thing per thing
  - Unpredictable Epochs (#245)
    - Decentralized apps need to tolerate forks. This doesn't work with a linear epoch counter.
    - Proposal: make epoch pseudo-random, derived from the key schedule.
    - Raphael: I'm not sure the server can do its job of ordering commit messages
      - Server just needs to enforce there are no two commits on the same epoch. This is possible with this proposal.
    - Jonathan Lennox: 32bit IDs will occasionally see collisions.
      - Jonathan Hoyland: are collisions recoverable?? Are they preventable, or are they manufacturable?
    - Conclusion: will continue to discuss on list.

# MLS Protocol Open Questions
Presented by Raphael
- Threat model
  - TreeKEM considers the compromise of 1 group member
  - RTreeKEM considers the compromise of 2 or more group members
    - If Alice is compromised, and then publishes an update to her sibling, Bob, who is later compromised before publishing an update, everything between the two compromises is available to the attacker.
    - Bob shouldn't be passive in this way, but applications may want to avoid frequent updates to large groups.
    - Solution: DH w/ deltas (UPKE), adds freshness to direct path siblings as well.
    - Richard Barnes recommends punting on this for v1
- Server Assist
- Send from Outside
  - Let an external party encrypt messages to the group
  - Use cases
  - Calvin: what about removal?
  - Richard: two auth problems, the sender to the group, and the group to the sender
  - Conclusion: Raphael will make a PR

# More Issues
Presented by Richard Barnes
- MLS Exporter (#198)
  - Use MLS to generate group secrets needed by other applications
- Tree of Signatures (#253)
  - New member must trust welcome sender to accurately represent tree
  - Proposal adds signatures to tree nodes
  - Jonathan Lennox: what's the threat model
  - Raphael: this is important.
  - Conclusion: will proceed with signature on leaf containing hashes of parents.
- PSK (#251)
  - Defines a new operation that imports user-provided PSK into the key schedule
  - Jonathan Hoyland: support channel binding(?)
  - Conclusion: discuss on list
- CIKs in Leaves (#254)
  - Move CIKs into leaves. Provides opening to rotate signing keys in the future.
- Measurement & Simulation

# Next Interim:
NYC. Updates to come on the mailing list.

The end!
