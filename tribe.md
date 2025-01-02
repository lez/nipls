# NIP: Tribes

`draft` `hierarchy` `tribe` `curation`

This NIP defines a standard for building social hierarchies - called tribes. A tribe manages its memberships and collectively curates content.

The aim is to be able to build culturally coherent groups at scale. Hierarchies are known to be a battle-tested solution to this.

Everything is public and plaintext: the structure, the content, the disputes and the decisions. Personal privacy is achieved via using pseudonyms.

Depending on the application, content curation can be just an abstract name for group moderation. These terms are used interchangedly in this NIP.

This is a client-based approach to build groups, though relays may offer services to tribe members. Tribes can move between relays freely.

This NIP does not support adding score / confidence numbers to relationships, or any kind of ranking. It operates solely with boolean decisions, aka curations. Something or somebody is either in or out.

### Building the tribe

A tribe is created by a lead, who is inherently a member of it. Tribes are hierarchical, members can add other pubkeys to the level below them by issuing stamps that reference their pubkeys.

The result is a tree graph, which is rooted at the lead. Each member can show up a trail of valid stamps that leads to them.

There are usecases where this is all we need. For example when we want to control who can write to a relay / media server.

A tribe can exist across multiple clients that support this NIP.

### Content Curation

A tribe may produce some kind of curated content, just like a media company produces a newspaper.

The curated content is made up of what the tribe members post. Additionally, individual events can be approved / disproved using kind:78 stamps that reference the event ids.

Direct moderation without previous notice should be left for spammers and abusers. If there is a chance for good intentions, kind:1984 reporting should be preferred where conflicts can be resolved.

The scope of the content curation (clients) should be specified by the tribe lead. This means you can still post anything as kind 1 note, but in the wiki app you have to follow your tribe's guidelines and other set of rules may apply in the internal chat.

## Technical details

### Stamps
There are two kinds:
```js
{
  "kind": 77,  // pubkey stamp
  "tags": [
    ["p", "<stamped-pubkey>"],
    ["c", "<context>"],
    ["nontransitive"],  // The stamped pubkey cannot stamp other pubkeys.
    ["revoked"],
    ["parent", "<event-id>"[, "<event-id>"...]],  // hints for pubkeys in levels above
    ...
  ],
  "pubkey": "<stamper-pubkey>",
  ...
}
```

```js
{
  "kind": 78,  // event stamp
  "tags": [
    ["e", "<stamped-event-id>"],  // multiple e-tags are supported
    ["c", "<context>"],
    ["revoked"],
	  ["parent", "<event-id>"[, "<event-id>"...]],  // hints...
    ...
  ],
  "content": "<reason>",
  "pubkey": "<stamper-pubkey>",
  ...
}
```

The context `"c"` describes what the stamp is valid for.
It is string that uniquely identifies the tribe. Either an event id, relay pubkey, even a tribe name is fine.

Pubkey stamps are transitive, unless the `["nontransitive"]` tag is set.

All members can stamp individual events, which brings a non-member pubkey's event into the curation.

A stamp can be revoked by the stamper by adding the `["revoked"]` tag. A revoked stamp invalidates a previously issued stamp. If it's a revoked event stamp, it moderates out a member's content.

Stamp events are transitive by default, which encourages taking on responsibilities. To maintain good quality of content, 1984 reporting/flagging of inappropriate behavior should be an integral part of keeping the conversation civil.

For cases where this process is too slow or the violation is obvious, shorter stamp trail beats the longer one. That means you can override stamps if you are higher up in the hierarchy. This opportunity should be used wisely and responsibly, only when needed, with clear reasoning.

For each stamper/stamped/context combination only the latest stamp event counts. Thus, it behaves as if it was a replaceable event with a `"d"` tag of `stamped/context`. But it's simpler this way.

If a member posts inappropriate content, it can be moderated out with a revoked event stamp. The revoked stamp must be originated from at least one level above the member. An event stamp is always stronger than a pubkey stamp that points to the same event and originated from the same level.

Additional machine readable metadata MAY be added to stamps as extra tags or any human readable text can be put into `content`.

### Relays

This NIP only specifies where the stamp event kinds are stored.
The client specific event kinds are stored based on the client's own logic.

Any client specific event should be acompanied by the stamp trail that prove its place in the tribe so as the relay can respond to tribe membership REQ's.

Stamps SHOULD also to be found on the read relays of the stamped pubkey or the author of a stamped event. (Or maybe just a notification?). This is to keep users in the flow of what's happening with their data.

Revoked stamps MAY be stored on its author's write relays so that anyone can check if a stamp is revoked. However, all the stamps SHOULD be available on the stamper's write relays for listing.

## Querying

These are the requests a client has to do in order to find out if an event is in or out.

0. Fetch the whole hierarchy recursively from the lead.
1. Fetch the client-specific events.
2. For each event, fetch the event stamps, which were issued for them.
3. Display or hide the event(s) according to the cached hierarchy and the fetched event stamps.

The interpretation of these stamps MUST be identical in order to be able to use it accross applications / services.
It should be clear and easy to tell if a pubkey is a member or an event is curated.

Right now the semantics of stamps are hard-coded in this NIP and the corresponding code. Later a scripting language can be built to define custom membership / curation rules (e.g. 2 distinct stamp trails are required for members)

## Notes

When a transitive pubkey stamp is revoked, the whole sub-tree of curation might disappear. Clients should warn users of this before revoking the stamp and offer a way to re-stamp the pubkeys and content down the hierarchy, and previously created content of the pubkey whose stamp is being revoked.
