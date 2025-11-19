# 01 - Create a simple definition for a first service

In this exercise we'll create a very basic definition and see how that becomes
the heart of everything. [The simplest thing that could possibly
work](https://creators.spotify.com/pod/profile/tech-aloud/episodes/The-Simplest-Thing-that-Could-Possibly-Work--A-conversation-with-Ward-Cunningham--Part-V---Bill-Venners-e5dpts),
pretty much. After all, every technical solution to a business problem has a
well thought out model at its core.

## Start a new CAP Node.js project

👉 Within the `hands-on-with-cap-cds/` directory, initialise a new CAP project
and change to the directory that is created:

```bash
cds init simple \
  && cd $_ \
  && npm install \
  && cds watch
```

> The only reason we're running `npm install` is so that we'll get
> project-local instances of the CAP runtime meaning that the log output will
> be a little less verbose as the paths to the implementations and default
> files should be shorter and relative to the current directory.

This should emit something like this:

```log
creating new CAP project in ./simple

adding nodejs

successfully created project – continue with cd simple

find samples on https://github.com/capire/samples
learn about next steps at https://cap.cloud.sap

cds serve all --with-mocks --in-memory?
( live reload enabled for browsers )

        ___________________________

    No models found in db/,srv/,app/,schema,services.
    Waiting for some to arrive...

```

The CAP server is started but is telling us (correctly) that there are no
models defined in any place it expects, so will not start listening for any
incoming requests as there is nothing to wrap a service around.

> The words "model" and "service" are chosen specificially and their meaning
> and distinction will become clear later on.

## Define a simple domain model

👉 Add the following content to a new file in the `simple/` directory called `services.cds`:

```cds
service Simple {
  entity Products {
    key ID    : Integer;
        name  : String;
        stock : Integer;
  }
}
```

> Technically we're combining a model definition inside a simple service
> definition here, but again, that distinction is for later.

As we've started the CAP server in [watch
mode](https://cap.cloud.sap/docs/tools/cds-cli#cds-watch), it should notice
these changes and restart, and this time, the log output includes, amongst
others (which have been redacted to keep things simple), these extra lines:

```log
[cds] - loaded model from 2 file(s):

  services.cds
  node_modules/@sap/cds/srv/outbox.cds

[cds] - connect to db > sqlite { url: ':memory:' }
/> successfully deployed to in-memory database.

[cds] - serving Simple {
  at: [ '/odata/v4/simple' ],
  decl: 'services.cds:1',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}

[cds] - server listening on { url: 'http://localhost:4004' }
```

> From the `odata` component of the relative URL path (`/odata/v4/simple`) we
> can correctly surmise that, by default, services such as this are made
> available in OData form, which is often going to be exactly what we need to
> provide a service to support an extension or new app to enhance our
> enterprise capabilities.

## Add some initial data

The `Simple` service is fully functional as the CAP framework provides
everything for a complete CRUD (Create, Read, Update, Delete) out of the box.
But let's add some data to make it a little easier to explore.

👉 Use the `data` facet of `cds add` to have a CSV file with a header line
added for the entities (just `Products` in this simple setup):

```bash
cds add data
```

The log output from this:

```log
adding data
adding headers only, use --records to create random entries
  creating db/data/Simple.Products.csv

successfully added features to your project
```

tells us where file is.

👉 Open the file and add the following records to it, after the header line:

```csv
1,Chai,39
2,Chang,17
3,Aniseed Syrup,13
```

---

[Next](../02/)
