---
trigger: always_on
description: This document describes how the databrowser stores your data as triples in RDF documents on your pod.
---

# Data Conventions used by the databrowser

This document describes how the databrowser stores your data as triples in RDF documents on your pod.
As most Solid pods will have at least some data on them that was edited through the databrowser,
these conventions are a good place to start when you develop third-party apps. In the future, we'll
see how this document can grow into a bigger participatory wiki of data shape conventions for Solid.

For short-hand, we will use the following namespace prefixes here:

```turtle
@prefix     ab: <http://www.w3.org/ns/pim/ab#> .
@prefix    acl: <http://www.w3.org/ns/auth/acl#> .
@prefix     dc: <http://purl.org/dc/elements/1.1/> .
@prefix    dct: <http://purl.org/dc/terms/> .
@prefix   flow: <http://www.w3.org/2005/01/wf/flow#> .
@prefix   foaf: <http://xmlns.com/foaf/0.1/> .
@prefix   ical: <http://www.w3.org/2002/12/cal/ical#> .
@prefix    ldp: <http://www.w3.org/ns/ldp#> .
@prefix    mee: <http://www.w3.org/ns/pim/meeting#> .
@prefix    pim: <http://www.w3.org/ns/pim/space#> .
@prefix    rdf: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix schema: <http://schema.org/> .
@prefix   sioc: <http://rdfs.org/sioc/ns#> .
@prefix  solid: <http://www.w3.org/ns/solid/terms#> .
@prefix   stat: <http://www.w3.org/ns/posix/stat#> .
@prefix     ui: <http://www.w3.org/ns/ui#> .
@prefix  vcard: <http://www.w3.org/2006/vcard/ns#> .
@prefix    XML: <http://www.w3.org/2001/XMLSchema#> .
```

One of the most important RDF documents on your pod is your profile, which is the document that people get when they dereference your webid. We'll look at that first. After that, we'll look at each of the tools that can be created with the databrowser's + button: Addressbook, Notepad, Chat, LongChat, Meeting, Event, Link, Document, Folder, and Source.

## Profile
(see also [pdsinterop.org's description](https://pdsinterop.org/conventions/profile/))

### Profile document

To add information to your webid profile, you can use the following triples. Suppose your webid is `/profile/card#me`, then your profile document is `/profile/card` (without the `#me`). Add the following triples to it:

```turtle
</profile/card> a                 foaf:PersonalProfileDocument .
</profile/card> foaf:maker        </profile/card#me> .
</profile/card> foaf:primaryTopic </profile/card#me> .
```

### You as a person

Now say your name is "John Doe", then add these triples to your profile document to publish your identity as a person:

```turtle
</profile/card#me> a         foaf:Person .
</profile/card#me> a         schema:Person .
</profile/card#me> foaf:name "John Doe" .
```

### Linking to your pod

Say your pod is at `/pod`, with the LDN inbox at `/pod/inbox/`, to link from your identity to your pod:

```turtle
</profile/card#me> solid:account </pod> .
</profile/card#me> pim:storage   </pod> .
</profile/card#me> ldp:inbox     </pod/inbox/> .
```

### Preferences

To publish some of your generic preferences to apps, use:

```turtle
</profile/card#me> pim:preferencesFile    </settings/prefs.ttl> .
</profile/card#me> solid:publicTypeIndex  </settings/publicTypeIndex.ttl> .
</profile/card#me> solid:privateTypeIndex </settings/privateTypeIndex.ttl> .
```

## Address Book
(see also [pdsinterop.org's description](https://pdsinterop.org/conventions/addressbook/))

You can create an addressbook containing persons and groups, by adding triples to RDF documents on your pod.
To create an addressbook, create a document for it, e.g., `/address-book/index.ttl`, and add the following triples to that document:

```turtle
</address-book/index.ttl#this> a         vcard:AddressBook .
</address-book/index.ttl#this> dc:title  "New address Book" .
</address-book/index.ttl#this> acl:owner </profile/card#me> .
```

You can create separate documents for the people index and for the groups index, as long as you link to those from the main `/address-book/index.ttl` document in the following ways:

```turtle
</address-book/index.ttl#this> vcard:nameEmailIndex </address-book/peopleIndex.ttl> .
</address-book/index.ttl#this> vcard:groupIndex     </address-book/groupIndex.ttl> .
```

To indicate that a person `/johnDoe.ttl` with full name "John Doe" is in addressbook `/address-book/index.ttl`, add the following triples:

```turtle
</johnDoe.ttl#this> vcard:inAddressBook </address-book/index.ttl#this> . # (NB: needs to be in /address-book/peopleIndex.ttl)
</johnDoe.ttl#this> a                   vcard:Individual .
</johnDoe.ttl#this> vcard:fn            "John Doe" .
```

To indicate that addressbook `/address-book/index.ttl` has a group called "Colleagues", add the following triples:

```turtle
</address-book/index.ttl#this>      vcard:includesGroup </address-book/colleagues.ttl#this> . # (NB: needs to be in /address-book/groupIndex.ttl)
</address-book/colleagues.ttl#this> a                   vcard:Group .
</address-book/colleagues.ttl#this> vcard:fn            "Colleagues" .
```

## Notepad
(see also [pdsinterop.org's description](https://pdsinterop.org/conventions/notepad/))

To create a new notepad at `/notepad.ttl`, add the following triples into it:

```turtle
</notepad.ttl#this> a          pim:Notepad .
</notepad.ttl#this> dc:author  </profile/card#me> .

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SolidOS/solid-panes](https://github.com/SolidOS/solid-panes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
