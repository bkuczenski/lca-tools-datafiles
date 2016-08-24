# lca-tools-datafiles
Files that describe available LCA data resources

[Visit the webpage](http://bkuczenski.github.io/lca-tools-datafiles/) for more information

## What is this about?

This is a repository for storing _catalogs_ of life cycle inventory (LCI) data used for preparing life cycle assessment (LCA) studies.  The catalogs are meant to describe the _semantic_ content of the databases, as well as the network structure, without describing the _quantitative_ information.  In other words, the catalogs include exchanges but not exchange values.

All the catalogs here are generated from publicly available databases using the [lcatools](http://github.com/bkuczenski/lca-tools/) python package (not yet registered anywhere!)

## What Databases are listed?

Currently, the catalog includes archives of the following data sources:

 * US LCI, via Ecospold v1 archive downloaded from [the LCA Commons](http://lcacommons.gov/nrel);
 * Ecoinvent v3.2 (four different system models), via the "activity overview" spreadsheets available on [the Ecoinvent website](http://www.ecoinvent.org/support/documents-and-files/information-on-ecoinvent-3/information-on-ecoinvent-3.html);
 * Ecoinvent LCIA factors, via the LCIA Implementation spreadsheet available from the ecoinvent Centre;
 * GaBi Professional database (2016), via their [web-based ILCD archive](http://www.gabi-software.com/support/gabi/gabi-database-2016-lci-documentation/professional-database-2016);
 * GaBi 2016 Extensions;
 * ELCD 3.2, via their [downloadable ILCD archive](http://eplca.jrc.ec.europa.eu/ELCD3/datasetDownload.xhtml)
 * ELCD LCIA factors from the same archive (only includes characterization factors for flows that actually appear as exchanges in ILCD)
 
Current plans include indexing the 2016 versions of GaBi Professional, GaBi Lean, and all 15-odd extension databases.  Others may come soon.

To download the catalogs, either clone this repository or visit the [Catalogs page](http://bkuczenski.github.io/lca-tools-datafiles/catalogs).

## How do I use it?

The python programming language makes it very easy to work with JSON data.  For a simple example of something useful to do, see [this notebook](https://github.com/bkuczenski/lca-tools-datafiles/blob/gh-pages/doc/Generate%20Manuscript%20Tables.ipynb).

# Data Model

The data model emphasizes **minimality** and **generality**.  The unifying characteristic is the following _minimal model for LCA_, which is an attempt at synthesizing a number of published LCA data models into a common model with the fewest possible parts:

![Minimal LCA Data Model](img/lca-model.png)

The model has three different entity types: `processes`, `flows`, and `quantities`.  

Each entity has **four** required fields: 

 1. an **entity type**;
 2. an **id** which uniquely identifies the data set within the catalog;
 3. an **origin**, which reports the original data source;
 4. an **external id**, which uniquely identifies the data set within the original data source.

In addition to those identity fields, each entity includes *descriptive* fields which contain semantic content, such as **Name** and **Comment**.  Processes and Flows also contain *structural* fields which link entities to one another. Processes contain a list of `exchanges`, which link one `process`, one `flow`, and a *direction* (`Input` | `Output`) of the flow with respect to the process.  Similarly, flows contain a list of `characterizations`, each of which is a pairing of a flow with a quantity, optionally including a locale.  Both exchanges and characterizations may include numerical exchange values, but they do not have to.  Both exchanges and characterizations can be labeled with the `isReference` property, which establishes that the exchange or characterization listed is the designated reference for that entity.
 
 * a `quantity` has a reference `unit` (units are outside the scope of this model)
 * a `flow` has a reference `characterization`
 * a `process` has a reference `exchange`.
 
Although Strictly speaking the reference object is supposed to be required, in practice the reference object is allowed to be `None`/ `null` / empty.
 
In addition to the mandatory fields, each entity has an unbounded collection of `tags` which can be any text-based or numeric content that describes the entity.  Processes are required to have 

 * "Name" (common to all entities)
 * "Comment" (common to all entities)
 * "SpatialScope" and "TemporalScope" (all `processes` in the catalogs have these)
 * "Compartment" (a hierarchical list; all flows have compartments)
 * "CasNumber" (all `flows` have this field, though it may be blank)
 * "UnitConversion" (some `quantities` have these)
 
Entities can also have any other tag.  The tags are meant to be fodder for search.

## JSON Archives

This repository contains archives and catalogs formatted as [[JSON]].  The JSON archivs have the following structure:

```
Archive = {
  "processes" : [ <list of process entities> ],
  "flows" : [ <list of flow entities> ],
  "quantities": [ <list of quantity entities> ],
  "dataSourceReference": "a string describing where the archive comes from",
  "dataSourceType": "a string describing the python subclass that created the archive"
}
```

Meanwhile, the entities themselves are simply defined:

`quantities` look like this:
```
quantity = {
  "entityId": quantity-identifier,
  "entityType": "quantity",
  "origin": dataSourceReference,
  "externalId": origin-specific identifier,
  "Name": "name text...",
  "Comment": "...",
  "referenceUnit": unit-identifier,
    ...
  }
}
```

`flows` look like this:

```
flow = {
  "entityId": flow-identifier,
  "entityType": "flow",
  "origin": dataSourceReference,
  "externalId": origin-specific identifier,
  "Name": "name text..."
  "Comment": "...",
  "CasNumber": "...",
  "Compartment": [ "...", "..." ... ],
    ...
  "characterizations": [
     {
       "entityType": "characterization",
       "quantity": quantity-identifier,
	...
     }
     ...
     ]
  }
},
```
`processes` look like this:

```
process = {
  "entityId": process-identifier
  "entityType": "process",
  "origin": dataSourceReference,
  "externalId": origin-specific identifier,
  "Name": "name text..."
  "Comment": "...",
  "SpatialScope": "spatial text...",
  "TemporalScope": 
  {
    "begin": validFrom,
    "end": validThrough
  },
  "exchanges": [
    {
      "entityType": "exchange",
      "flow": flow-identifier,
      "direction": "Input" | "Output",
      ...
    }
    ...
    ]
  }
}
``` 

In each case, the identifier listed in one entity's `entityId` field should match the identifier listed in an
exchange or characterization in which it is referenced.

`<direction>` is either `Input` or `Output`.

## Data Source Types

Within each archive, the `externalId` fields should each uniquely identify an entity with respect to the `origin` value, but interpretation varies depending on the archive type. For instance, if the `dataSourceType` is an `IlcdArchive`, then the `dataSourceReference` points to the root of the archive and the `externalId` would be things like `processes/<uuid>` or `flowproperties/<uuid>`.

On the other hand, if the archive is an `EcospoldV1Archive`, then `processes` are identified by name, `flows` are identified by an internal ID integer, and `quantities` are just unit strings.

Finally, if the archive is an `EcoinventSpreadsheet`, then `processes` are UUIDs; `flows` are uniquely identified by name if they are intermediate flows, or uniquely identified by name, compartment, and subcompartment if they are elementary flows; and `quantities` are-- again-- just unit strings.

The internal `entityId` is always simply a UUID.

