# MLS IETF 114

Outline
* Administrivia (10min)
* Note Well
* Note Taker - Paul Wouters
* Jabber Scribe
* Status
2x almost WGLC, 2x close
* Protocol/Architecture I-Ds (20min)
This agenda item is only needed if there are late WGLC comments/issues/prs or timely AD comments.

## Protocol
Datatracker: https://datatracker.ietf.org/doc/draft-ietf-mls-protocol/
GitHub: https://github.com/mlswg/mls-protocol

## Architecture
Datatracker: https://datatracker.ietf.org/doc/draft-ietf-mls-architecture/
*GitHub: https://github.com/mlswg/mls-architecture

## Federation - Raphael Robert (30min)
I-D:
Datatracker: https://github.com/mlswg/mls-federation
GitHub: https://github.com/mlswg/mls-federation
Slides: TBD

## Extensions - Raphael Robert (30min)
I-D:
Datatracker: https://github.com/mlswg/mls-extensions
GitHub (coming soon): https://github.com/mlswg/mls-extensions
Slides: TBD
* MLS Extensions can go into its own smaller documents.
* Sean: IESG prob prefers no blank check Extension document.
* Nobody in the room read the draft.

# Content Negotiation - Rohan Mahy (20min)
I-D:
Datatracker: https://datatracker.ietf.org/doc/draft-mahy-mls-content-neg/
Slides: TBD
* Nobody in the room read this document either
* Which application formats are required and/or supported in (long lived) group
* No builtin way to signal
* Proposes MLS extension(s)
* Ted Hardie: maybe constrain accepted media types, eg not model or font
* dkg: what Ted said. multi-part alternative, implies mime. how does that relate to mime ? multi-part is scary top level you might wand to avoid
* Paul Wouters (individual): the "first media type is assumed" sounds like a dangerous construct, if it is an unused first entry triggered 10 years after its use by an attacker. Maybe make media_type just mandatory?
* Ted: (some talk about media type registrations and that this is hard). Maybe be strict at first, then open up IANA Registry later on.
* Sean (chair): kind of out of scope of chater, but seems needed. Talk to AD.
* Richard BarneS: would be nice if we can use ALPN
* Raphael: we might lack expertise in this WG.
* Ted: maybe to MIMI WG to be?
* Paul (AD): Lets ensure to work on core docs first, then we have some time to see where best to land this.
