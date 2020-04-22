April 21, 2020

# Incoming Pre-shared Key and Ciphersuites PR

Britta, Konrad

Britta: Currently we just have PSK injection. Going to define how to use that
feature for different purposes. Four cases that need to be handled separately:
re-initialization, recovery from state loss, integrating an external PSK, and
sub-group branching.

Konrad: High-level changes:
- For re-initialization: add Re-Init proposal.
- For recovery: have way to indicate when new group is created, that
  recovery_key from previous group must be known.
- For branching: Unclear.
- For external PSK: add External-PSK proposal.

Konrad: Concrete changes to key schedule:
- Flip order of PSK and commit_secret injection.
- Add derivation of a recovery_secret.

Richard: Why are these changes against the main key schedule instead being
implemented with exporter keys?

Konrad: We wanted to keep the functionalities separate.

Britta: It's cleaner to keep what's internal to the group, internal.

Joel: What's the difference between recovery and re-initialization due to a new
ciphersuite?

Konrad: Recovery assumes the latest epoch is corrupted. Re-initialization
doesn't.

Raphael: Would be nice to use re-init as an upgrade mechanism.

Richard: Recovery mode seems too heavyweight. We want to fix a few people, not
restart everybody. Don't understand branching, I don't think anybody would want
it.

Konrad: I'll elaborate on branching: Say you have one big group for a company,
and you want to branch into subgroups that inherit the security of the big
group. You'd do this so, if my key is compromised, I can Update in the main
group and then all the branched groups are secure again as well.

Chris Wood: Would be good for PSK efforts in MLS and TLS not to diverge too
much.

Raphael: I believe Jon Millican was looking into a recovery mechanism, ACKS /
NACKS. Should check-in with him.

Richard: Should mix this in with resolving #325.


# 325: Simplify epoch secret derivation

Richard: Not sure about reducing hash invocations.

Nick: Consensus on this issue?

Britta: We're going to roll this into a bigger discussion.

Richard: We need a dedicated call on key schedule revisions.

**DECISION:** Schedule interim call specifically for talking through key
schedule issues.


# 301: Targeted message

Richard: This doesn't require any key schedule re-arrangements, we're just
deriving some new things?

Raphael: Yes, probably doesn't involve key schedule changes. Need a safe way to
KEM messages to individual leaves.

Richard: I see, this overlaps with the branching case.

Raphael: Well, when you branch you create a new MLS group, and this doesn't
require that. To solve this with branching, we'd need to make sure you keep the
groups in sync.

Richard: It's possible to implement this using exporter keys and leaving it to
the application. How important is it to have this in the base spec?

Raphael: You'd have to think about how bad it could be if application's need to
figure this out themselves.

Konrad: I think it'd be good to have in the base spec, as it would also help
with federation and is a more-or-less standard thing applications will want.

Raphael: Yes, we use targeted messages for read receipts and call signaling.

**DECISION:** Save targeted messages for the big PSK / key schedule interim
call.


# 302: Use masking instead of AES-GCM for sender data

Brendan: How would it work?

Chris Wood: In QUIC, there's a separate key for metadata. We take a sample of
the encrypted ciphertext and compute a PRF of it under that key. You XOR the PRF
output with the data you want to encrypt.

Joel: So it's malleable.

Richard: Yes. There's a paper that proves the security. There's a circular
dependency and if anything is corrupted, decryption fails.

Chris Wood: There's discussion in the QUIC working group about pulling this
construction out and making a CFRG draft. Would be nice if we could just cite
that draft.

Richard: I was thinking we'd just copy it in now and then the CFRG draft would
have two instances to reference.

Brendan: Wouldn't this make ciphertexts deterministic, since sender data nonce
is the only randomness?

Richard: We still have the reuse guard which is 4 bytes. We'd be leaning on it
more heavily.

**NOTE:** This obsoletes #308 if accepted.


# 317: Change expiration extension to lifetime extension.

Richard: Lgtm.


# 320: Use node type instead of leaf index in tree hashes.

Brendan: Removing the additional data has no impact on security and makes
implementations simpler.

Richard: It's not complicated to compute the information you need.

Joel, Konrad: Having the extra information simplifies the proofs.

**ACTION ITEM:** Implement fix suggested in #328.
