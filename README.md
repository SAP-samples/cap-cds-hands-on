[![REUSE status](https://api.reuse.software/badge/github.com/SAP-samples/cap-cds-hands-on)](https://api.reuse.software/info/github.com/SAP-samples/cap-cds-hands-on)

# Hands-on with CAP CDS

Gain a thorough grounding in CAP CDS modelling.

## Introduction

In this hands-on workshop you'll become familiar and comfortable with the key
features of CDS modelling, primarily focusing on the Conceptual Definition
Language ([CDL](https://cap.cloud.sap/docs/cds/cdl)), the predominantly
declarative domain-specific language (DSL) in the [CDS
family](https://cap.cloud.sap/docs/cds/). You'll explore the different concepts
that make CAP such a compelling framework for today's world of AI, where
concision, correctness and the imperative to write less code is at the
forefront of everyone's mind.

No prior experience is required, and this workshop is suitable for
non-developers as well as developers. All you need is curiosity and
a desire to learn!

This workshop was originally written for a hands-on session at UKISUG Connect
2025 (see [Hands-on domain modelling with CAP's CDS at UKISUG
Connect](https://qmacro.org/blog/posts/2025/11/11/hands-on-domain-modelling-with-caps-cds-at-ukisug-connect/)),
but has now been significantly extended.

## Prerequisites

In order to work through the exercises, you'll need a development environment
for CAP Node.js. See the [prerequisites info](prerequisites/README.md) page for
details and options.

The exercises presume no prior knowledge; nor do they attempt to cover
everything there is to know about CDS modelling and CDL. For that, see the
relevant sections of [Capire](https://cap.cloud.sap/docs), particularly the
[CDS](https://cap.cloud.sap/docs/cds/) topic.

## Exercises

To get started, clone this repository and open it in your favourite editor or
IDE. Alternatively, launch a GitHub Codespace directly from this repository -
see the [prerequisites] for more info.

### Part 1 - Understanding the context and some basic definitions

When, where and why does one use CDL? To define CDS models that reflect the
problem domain, the business entities that make up the solution landscape. Who
is responsible for this? Teams of developers and business domain experts combined;
between them the domain knowledge can be accurately expressed and modelled as the
foundation for a service, solution or application.

The exercises in this part help us understand better the context of domain
modelling with CDS.

- [01 - Create a simple definition for a first service](exercises/01/)
- [02 - Understand the basic model, service and persistence features](exercises/02/)
- [03 - Separate out the data model from the service definition](exercises/03/)

#### Related resources for part 1

- The [What is CAP?](https://cap.cloud.sap/docs/about/#what-is-cap) and
  [Jumpstart & Grow As You
  Go](https://cap.cloud.sap/docs/about/#jumpstart-grow-as-you-go) sections of
  the Getting Started topic in Capire
- [Exercise 01 - cds watch, SQLite, initial data and sample
  data](https://github.com/SAP-samples/cap-local-development-workshop/tree/main/exercises/01)
  of the [CAP local development
  workshop](https://github.com/SAP-samples/cap-local-development-workshop)
  material, for more on initial vs sample data
- The [Keep services
  simple](https://qmacro.org/blog/posts/2026/04/07/cds-expressions-in-cap-notes-on-part-5/#keep-services-simple)
  section of the - notes on Part 5 of the CDS expressions in CAP series
- Axiom [AXI004 Services are cheap](https://github.com/qmacro/capref/blob/main/axioms/AXI004.md)

### Part 2 - More on structure with types, aspects and reuse

The definition we have so far is deliberately very basic. What facilities in
CDL are available to expand on that? What about the definition of custom types
to bring consistency and at the same time avoid repetition? Perhaps most
importantly, how can we define our domain models in a way that reuse is always
possible, both in and of what we are building?

In this part we'll expand our basic definitions as a way of learning about the
answers to these questions.

- [04 - Design and use custom types for a richer entity definition](exercises/04/)
- [05 - Explore reuse and standard common definitions](exercises/05/)
- [06 - Understand and use aspects](exercises/06/)

#### Related resources for part 2

- The [Language
  Preliminaries](https://cap.cloud.sap/docs/cds/cdl#language-preliminaries)
  section of the CDL topic in Capire
- The sections on type definitions and structured types in the [Entities & Type
  Definitions](https://cap.cloud.sap/docs/cds/cdl#entities-type-definitions)
  section of the CDL topic in Capire
- The section on [Aspects](https://cap.cloud.sap/docs/cds/cdl#aspects) in the
  CDL topic in Capire
- CAP related [blog posts that talk about
  aspects](https://qmacro.org/tags/aspects/)
- The blog post [ISO content for common CAP
  types](https://qmacro.org/blog/posts/2024/03/12/iso-content-for-common-cap-types/)
  that describes and demonstrates the use of an NPM package that provides
  default content based on the ISO specifications for CAP common reuse types
  for countries, languages, currencies and timezones
- The blog post [Modelling contained-in relationships with compositions in
  CDS](https://qmacro.org/blog/posts/2025/10/14/modelling-contained-in-relationships-with-compositions-in-cds/)
  which talks about the use of anonymous aspects
- The blog post [Flattening the hierarchy with
  mixins](https://qmacro.org/blog/posts/2024/11/08/flattening-the-hierarchy-with-mixins/)
  on the advantages of embracing aspect oriented programming techniques.

### Part 3 - Describing relationships with associations and compositions

At this point in the workshop we have a couple of entities, representing
products and suppliers. But they're completely separate from one another, with
no relation between them.

In this part we'll look at the facilities in CDL for describing relationships,
and add a conjoined pair of entities to see how they are are manifested and behave.

- [07 - Link entities together with associations](exercises/07/)
- [08 - Define contained-in relationships with compositions](exercises/08/)
- [09 - Try out deep inserts and cascaded deletes](exercises/09/)

### Part 4 - Exposing models via services - interfaces for the outside world

Thus far the vast majority of work, and all of the focus, has been at what we
understand by now to be the `db/` layer - the core entity definitions and
relationships between them. While we've dabbled briefly with a service
definition, that was just a means to an end, to allow us to look at our model
constructions through the lens of the OData V4 standard.

In this part we'll turn our focus to the `srv/` layer and look at why it's
separate and start to explore what we can do there.

- [10 - Explore projections with a second service](exercises/10/)
- [11 - Take a first look at domain specific custom operations](exercises/11/)
- [12 - Add a further operation in the form of a bound action](exercises/12/)

#### Related resources for part 4

- The [Providing
  Services](https://cap.cloud.sap/docs/guides/providing-services) topic in
  Capire
- A two minute video on [HTTP, the HyperText Transfer
  Protocol](https://www.youtube.com/watch?v=Ic37FI351G4)
- A six-part [Back To Basics series on
  OData](https://www.youtube.com/playlist?list=PL6RpkC85SLQDYLiN1BobWXvvnhaGErkwj)
- [A detailed look at OData
  metadata](https://github.com/qmacro/odata-dd-tutorials/blob/main/tutorials/odata-dd-4-metadata/odata-dd-4-metadata.md),
  part of the sources for the tutorials in the [OData Deep Dive
  mission](https://developers.sap.com/mission.odata-deep-dive.html)
- An [in-depth series looking at the CDS Expression Language
  (CXL)](https://qmacro.org/blog/posts/2025/12/09/a-new-hands-on-sap-dev-mini-series-on-the-core-expression-language-in-cds/)

### Part 5 - Annotations and solid state programming

With the decent grounding in CDS modelling that we have at this point, we can
turn to more powerful features that make CAP the powerful, declarative and
concise framework that it is.

- [13 - An introduction to annotations](exercises/13/)
- [14 - Declarative constraints and assertions](exercises/14/)
- [15 - Shift left, avoid procedural code](exercises/15/)

#### Related resources for part 5

- The tutorial [Understand how vocabularies are used in OData metadata
  documents](https://developers.sap.com/tutorials/odata-dd-5-vocabularies.html),
  relating to the alternative annotations for `@title` and `@description`
  examined in exercise 13
- The blog post [A deep dive into OData and CDS
  annotations](https://qmacro.org/blog/posts/2023/03/10/a-deep-dive-into-odata-and-cds-annotations/)
  has lots of info on how to read and understand such things
- See the live stream series on the CDS Expression Language for lots of detail
  and deep diving into the topic: episode replays and accompanying notes are
  available via the blog post [A new Hands-on SAP Dev mini-series on the core
  expression language in
  CDS](https://qmacro.org/blog/posts/2025/12/09/a-new-hands-on-sap-dev-mini-series-on-the-core-expression-language-in-cds/)
- The blog post [Constraints, expressions and axioms in
  action](https://qmacro.org/blog/posts/2026/01/27/constraints-expressions-and-axioms-in-action/)
  has more on declarative contraints.

## Support

Support for the content in this repository is available during the actual time
of the workshop event for which this content has been designed.

## License

Copyright (c) 2025 SAP SE or an SAP affiliate company. All rights reserved.
This project is licensed under the Apache Software License, version 2.0 except
as noted otherwise in the [LICENSE](LICENSE) file.
