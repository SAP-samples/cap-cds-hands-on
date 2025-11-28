# 01 - Create a simple definition for a first service

In this exercise we'll create a very basic declarative definition and see how
that becomes the heart of everything ([the simplest thing that could possibly
work](https://creators.spotify.com/pod/profile/tech-aloud/episodes/The-Simplest-Thing-that-Could-Possibly-Work--A-conversation-with-Ward-Cunningham--Part-V---Bill-Venners-e5dpts),
pretty much). After all, every good technical solution to a business problem
has a well thought out model at its core.

## Start a new CAP Node.js project

👉 Within the `hands-on-with-cap-cds/` directory, initialise a new CAP project,
change to the directory that is created, and invoke a package install for the
requisite modules:

```bash
cds init simple \
  && cd $_ \
  && npm install
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
```

👉 Now start the CAP server up:

```bash
cds watch
```

This should emit something like this:

```log
cds serve all --with-mocks --in-memory?
( live reload enabled for browsers )

        ___________________________

    No models found in db/,srv/,app/,schema,services.
    Waiting for some to arrive...

```

The CAP server is started but is telling us (correctly) that there are no
models defined in any place it expects, so will not start listening for any
incoming requests as there is nothing to wrap a service around and serve.

> The words "model" and "service" are chosen specificially and their meaning
> and distinction will become clear later on.

## Define a simple domain model

For the rest of this exercise (and the other exercises in this part of the
workshop) you can remain in the `simple/` directory - the "project root" for
the CAP project we'll be building out as we work through the exercises. Any
relative reference to directories or files will be relative to this `simple/`
location.

👉 Add the following content to a new file called `services.cds`:

> The name of the file (`services.cds`) is important in this example, it is one
> of the default places the CAP server will look for definitions.

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

> [!NOTE]
> There are three new CDL keywords here:
>
> - `service` to declare a [service
>   definition](https://cap.cloud.sap/docs/cds/cdl#service-definitions), an
>   interface to the outside world, generally speaking
> - `entity` to introduce a [structured type that represents persisted
>   data](https://cap.cloud.sap/docs/cds/cdl#entity-definitions), in other
>   words, a business object (in the loosest sense)
> - `key` to indicate that the element is to be considered a primary key

As we've started the CAP server in [watch
mode](https://cap.cloud.sap/docs/tools/cds-cli#cds-watch), it should notice
these changes and restart, and this time, the log output includes, amongst
other info (which has been removed to keep things simple), these extra lines:

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

It has:

- found our definitions in `services.cds`
- established an in-memory SQLite mechanism
- started serving the service we've defined

> From the `odata` component of the relative URL path (`/odata/v4/simple`) we
> can correctly surmise that, by default, services such as this are made
> available in OData form, which is often going to be exactly what we need to
> provide a service to support an extension or new app to enhance our
> enterprise capabilities.

## Add some initial data

The `Simple` service is fully functional as the CAP framework provides
everything for a complete CRUD implementation (Create, Read, Update, Delete)
out of the box. But let's add some data to make it a little easier to explore.

👉 Use the `data` facet of `cds add` to have a CSV file with a header line
added for the entities (just `Products` in this simple setup):

> To enter this command, while the CAP server (started with `cds watch`) is running,
> you should start a new terminal session, and use the shell in that second session
> for such commands, not only here, but throughout the rest of the workshop. Note
> that when you open the second terminal, you'll be in the shell, but at the root
> of the repository, i.e. `cap-cds-hands-on/`, so make sure you move into the
> "project root" (with `cd simple/`) so you're invoking commands in the right place.

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

👉 Open the file and observe the CSV header line in there, which should look
like this:

```csv
ID,name,stock
```

👉 Append the following records, after the header line:

```csv
1,Chai,39
2,Chang,17
3,Aniseed Syrup,13
```

> Some of you may recognise the product names, they're from the classic
> [Northwind](https://services.odata.org/V4/Northwind/Northwind.svc/) sample
> service that now has almost legendary status (technically they're from a cut
> down version of Northwind, called
> [Northbreeze](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze)
> which is sometimes used in other workshops in this context).

In the section of the CAP server log that we saw before announcing the use of
SQLite, we now see an extra line telling us this CSV file has been found and
initial data is being loaded from it:

```log
[cds] - connect to db > sqlite { url: ':memory:' }
  > init from db/data/Simple.Products.csv
/> successfully deployed to in-memory database.
```

Great! Time to explore our fledgling design.

---

[Next](../02/)
