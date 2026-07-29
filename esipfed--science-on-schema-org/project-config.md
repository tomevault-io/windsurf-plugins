---
trigger: always_on
description: This document explains the conventions used within this guide.
---

# Conventions #

This document explains the conventions used within this guide.

* [Syntax](#syntax)
    * [General Rules](#syntax)
    * [JSON-LD Snippets](#syntax-snippets)
    * [Creating Diagrams](#syntax-diagrams)
    * [Creating Figures](#syntax-figures)
    * [Schema.org Namespace](#syntax-namespace)
* [Versioning](#versioning)
* [Why JSON-LD](#why-jsonld)
* [Typing to External Vocabularies](#external-vocab-typing)

<a id="syntax"></a>
## Syntax ##

1. Use **[JSON-LD](https://json-ld.org/)** in our guidance documents for simplicity and terseness as compared to *[Microdata](https://www.w3.org/TR/microdata/)* and *[RDFa](https://rdfa.info/)*.
2. Documents should start with:
  1. An named anchor of 'top': ```<a id="top"></a>```
  2. A breadcrumb trail respective to the level in the guide:  
  
     <pre>[Home](/README.md) | [Some directory](/guides/<dir-name>) | This Location in the guide</pre>

2. Documents should describe *how* and *why* for each class and property being recommended.
<a id="syntax-snippets"></a>
3. **JSON-LD snippets** should be wrapped within a preformatted text block using **&lt;pre&gt;&lt;/pre&gt;** tags. To highlight a section of JSON-LD, use **&lt;strong&gt;&lt;/strong&gt;** tags.

<pre>
{
  "property": "value",
  <strong>"highlighted-property": "highlighted-value",</strong>
}
</pre>

<a id="syntax-diagrams"></a>
4. **Creating Diagrams**

  1. When recommending which properties of a schema.org class to use, creating a diagram of the connections between those resources and literal values can be helpful for visualizing the big picture.
  2. Diagrams should follow the following convention:
  [![Graphical Notation](/assets/diagrams/graphical-notation.svg "Graphical Notation")](#)
  3. Diagrams are currently being made using [Lucidchart](https://www.lucidchart.com), and are being shared to edit here: [schema.org diagrams on Lucidchart](https://www.lucidchart.com/documents#docs?folder_id=170151578&browser=icon&sort=saved-desc)

<a id="syntax-figures"></a>
5. **Creating Figures**

  1. Save the image to the ```/assets``` directory within this guide,
  2. Then, in the document, decide where the figure should be added,
  3. Add a named anchor before the figure, ```<a id="figure-(x)"></a>```, where ```(x)``` is a unique, incremental number.
  4. Center the figure by wrapping it and it's text with: ```<p align="center">...</p>```
  5. The first element in the section should be the figure title: ```Figure (x). The figure title goes here...```
  6. Next, insert the image with: ```<img src="/assets/<path-to-image-file e.g. schemaorg-datatypes.png>">```
  7. Optionally, add an italicized description with: ```<em>optional description goes here...</em>```

Figure example: 
```
  <a id="figure-1"></a>
  <p align="center">
    <strong>Figure 1. schema.org field data types</strong><br/>
    <img src="/assets/schemaorg-datatypes.png">
    <em>The expected data type for each field appears in the middle column. The left column is the name of the field, the middle column is the data type, and the right column is the field's description.</em>
  </p>
```

<a id="syntax-namespace"></a>
6. **Namespace for `schema.org`.** 

We recommend that the namespace URI for the schema.org vocabulary be consistently set to the value `http://schema.org/`.

 Consistent representation of namespaces simplifies programmatic processing of schema.org markup. For example, even though conceptually it is clear that the terms `http://schema.org/Dataset` and `https://schema.org/Dataset` (note the protocol difference) are referring to [http://schema.org/Dataset](https://schema.org/Dataset), these are programmatically treated as different entities. The [schema.org guidelines](https://schema.org/docs/faq.html#19) are somewhat ambivalent on the topic. Their context files maintain an `http` namespace for all terms, but can also be resolved from `https`-based addresses.

While the schema.org context is resolvable at both the original `http` address and the newer `https` address, changing the namespace to `https` references would also effectively change the namespace of the terms to a new namespace. While schema.org maintainers have clarified that both namespaces have been actively recommended for 5 years, and are considered synonomous, the official context file that is retrieved from both `https://schema.org/` and `http://schema.org` define the namespace URI for the vocabulary as `http://schema.org/`. For consistency, we thus recommend using the `http`-based namespace so that term URIs stay comparable over time, but also that harvesters consumers treat the http and https namespaced terms as logical synonyms.

One consistent mechanism to use the `http`-based namespace is to load the context from the context file maintained by schema.org. This can be accomplished using:

```json
{
    "@context": "https://schema.org/",
    "@type": "Dataset",
    "name": "Example dataset title"
}
```

While that context is loaded from the `https` address above, the resulting JSON-LD fragment uses the `http` namespace when it is expanded:

```json
[
  {
    "@type": [
      "http://schema.org/Dataset"
    ],
    "http://schema.org/name": [
      {
        "@value": "Example dataset title"
      }
    ]
  }
]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ESIPFed/science-on-schema.org](https://github.com/ESIPFed/science-on-schema.org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
