# Hands-on with CAP CDS

A hands-on introduction to the key features of the Conceptual Definition
Language (CDL) used in CAP CDS modelling.

## Introduction

The content in this repo is designed for a two hour hands-on workshop that
introduces the key features of CDS modelling, focusing on the Conceptual
Definition Language ([CDL](https://cap.cloud.sap/docs/cds/cdl)), the
predominantly declarative domain-specific language (DSL) [in the CDS
family](https://cap.cloud.sap/docs/cds/).

The workshop abstract reads: "_Become acquainted with CAP's CDS, the common
language that binds domain experts, with their key business & process
knowledge, to developers. A hands-on-optional session where we'll explore the
language & concepts together and get comfortable with it. (More info is
available on the session detail page)_". For more information about the
workshop, see [Hands-on domain modelling with CAP's CDS at UKISUG
Connect](https://qmacro.org/blog/posts/2025/11/11/hands-on-domain-modelling-with-caps-cds-at-ukisug-connect/).

## Prerequisites

In order to work through the exercises, you'll need a development environment
for CAP Node.js. See the [prerequisites](prerequisites.md) page for details and
options.

The exercises presume no prior knowledge; nor do they attempt to cover
everything there is to know about CDS modelling and CDL. For that, see the
relevant sections of [Capire](https://cap.cloud.sap/docs), particularly the
[CDS](https://cap.cloud.sap/docs/cds/) topic.

## Exercises

To get started, clone this repository and open it in your favourite editor or
IDE.

```bash
git clone https://github.com/qmacro/hands-on-with-cap-cds \
  && cd $_
```

Unless otherwise stated, all command line invocations are based on being in the
clone directory, i.e. within `hands-on-with-cap-cds/`.

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

#### Further reading

- The [What is CAP?](https://cap.cloud.sap/docs/about/#what-is-cap) and
  [Jumpstart & Grow As You
  Go](https://cap.cloud.sap/docs/about/#jumpstart-grow-as-you-go) sections of
  the Getting Started topic in Capire.

### Part 2 - More on structure with types, aspects and reuse

The definition we have so far is deliberately very basic. What are the
facilities in CDL to expand on that, to allow for the definition of custom
types, to bring consistency and at the same time avoid repetition? Perhaps most
importantly, how can we define our domain models in a way that reuse is always
possible, both in and of what we are building?

In this part we'll expand our basic definitions as a way of learning about the
answers to these questions.

- [04 - Design and use custom types for a richer entity definition](exercises/04/)
- [05 - Explore reuse and standard common definitions](exercises/05/)
- [06 - Understand and use aspects](exercises/06/)

#### Further reading

- The [Language
  Preliminaries](https://cap.cloud.sap/docs/cds/cdl#language-preliminaries)
  section of the CDL topic in Capire
- The sections on type definitions and structured types in the [Entities & Type
  Definitions](https://cap.cloud.sap/docs/cds/cdl#entities-type-definitions)
  section of the CDL topic in Capire
- The section on [Aspects](https://cap.cloud.sap/docs/cds/cdl#aspects) in the
  CDL topic in Capire
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
definition on occasion, that was just a means to an end, to allow us to look at
our model constructions through the lens of the OData V4 standard.

In this part we'll turn our focus to the `srv/` layer and look at why it's
separate and what we can do there.

- [10 - Explore service definitions](exercises/09/)

#### Further reading

- The [Providing Services](https://cap.cloud.sap/docs/guides/providing-services) topic in Capire
