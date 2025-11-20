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

## Exercises

To get started, clone this repository and open it in your favourite editor or
IDE.

```bash
git clone https://github.com/qmacro/hands-on-with-cap-cds \
  && cd $_
```

Unless otherwise stated, all command line invocations are based on being in the
clone directory, i.e. within `hands-on-with-cap-cds/`.

### Part 1 - Understanding the context

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
