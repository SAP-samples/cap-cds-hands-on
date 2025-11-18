# 01 - The simplest thing that could possibly work

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

👉 Open the file and add the following records to it:

```csv
1,Chai,39
2,Chang,17
3,Aniseed Syrup,13
```

## Explore the service

👉 Open up `http://localhost:4004/` in your browser to see the default CAP
server landing page:

![Default CAP server landing page](assets/default-cap-server-landing-page.png)

👉 Examine some of the resources available via the various hyperlinks:

- [http://localhost:4004/odata/v4/simple](/odata/v4/simple) (the OData service
  document)
- [http://localhost:4004/odata/v4/simple/\$metadata](/odata/v4/simple/$metadata)
  (the OData metadata document)
- [http://localhost:4004/$fiori-preview/Simple/Products#preview-app](Fiori preview) (a basic Fiori elements List Report app where the Products data can be viewed)
- [http://localhost:4004/odata/v4/simple/Products](/odata/v4/simple/Products) (an entityset with the Products data)

👉 Riff on this last resource by trying out some standard OData URL-based query mechanisms (admittedly these are somewhat limited, given the rather limited nature of our dataset), such as:

- [http://localhost:4004/odata/v4/simple/Products/3](http://localhost:4004/odata/v4/simple/Products/3) (retrieve the Product with ID 3)
- [http://localhost:4004/odata/v4/simple/Products?\$filter=contains(name,%27Syrup%27)](http://localhost:4004/odata/v4/simple/Products?$filter=contains(name,'Syrup')) (return an entityset with products that have "Syrup" in their names)
- [http://localhost:4004/odata/v4/simple/Products?\$select=name](http://localhost:4004/odata/v4/simple/Products?$select=name) (all the products, just with the name property for each entity)

> By default the key property, in this case `ID`, is returned as well.

👉 At the shell prompt, add a new product:

```bash
lastid="$(
  curl -s 'localhost:4004/odata/v4/simple/Products?$orderby=ID%20desc&$top=1' | \
    jq '.value|first|.ID'
)" # get latest ID
nextid="$((lastid + 1))" # increment it
curl -d '{"ID":'"$nextid"',"name":"New Product ('"$nextid"')"}' \
  --url 'localhost:4004/odata/v4/simple/Products' # use for a new record
```

👉 Change the name of product with ID 3:

```bash
curl --request PATCH \
  --data '{"name": "Aniseed Sauce"}' \
  --url 'localhost:4004/odata/v4/simple/Products/3'
```

👉 Remove the "Chai" product:

```bash
curl --request DELETE \
  --url 'localhost:4004/odata/v4/simple/Products/1'
```
