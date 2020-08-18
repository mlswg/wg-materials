# MLS WG Meeting
2020-07-28, IETF108 - Virtual

# Note Well

# Issues/PRs

Richard Barnes provided an overview of the protocol-related (draft-ietf-mls-protocol) changes since the last meeting: leverage HPKE and TLS signatures, ratcheting optional for adds, and tree optional in GroupInfo message. The remaining time was spent discussions open issues:

* Reflecting reality in the key schedule: when a PSK is used, the PSK does not authenticate the new joiners know it and the GroupContext gets used in a bunch of individual derivations. Proposal is to reorder so that the joiner has to use the PSK to get the epoch secret and to add GroupContext into the epoch_secret once.

* Simplifying sender data encryption: the goal is to prevent the delivery service from seeing sender and generation. Proposal was to either swap order of content and metadata encryption but via jabber SIV was proposed. The discussion will be taken to the list.

* nKDF discussed on CFRG: Key scheduled was changed so it no longer clear that nKDF is need, i.e., no longer need multiple keys incorporated at once.
