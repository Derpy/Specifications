# PTEP-004 - Core Schema

The core schema defines the essence of how data will be represented 
and grow, it defines what kind of things are core objects, the kinds
of actions which will be performed on them and methods of retrieval.

## Semantics

We must first explain the sorts of complexity involved which need
to be right from the get-go and the semantic principles behind it.

### Products, Versions and Variations

One of the downfalls prevelent in the 'torrent industry' is the notion
of Versions and Variations of Products because the sites do not strive
to act like a Shop for their Customers, it becomes a basic inventory
keeping problem which has been successfully tackled for the good part
of a century just by looking at it from a different perspective.

For legal and semantic reasons the this terminology is being used, and
in the long-run it will keep design on-track.

 * Product - something which a Customer desires.

 * Version - chronological or technical difference.

 * Variation - e.g. different bitrate or paint colour.

There may be many versions and variations of versions of a product,
but there will only be one product.

Example:

```
  Product - BurnMaster
  Version - 9000
  Variation - Black, Extra Large
```

Additional Example:

```
  Product - Red Tails
  Versions - BRRip 2012
  Variation - x264 - 800MB - YIFY
```

To give an ideal use case, a request can be made for a Product or 
variation of one which does not yet exist in the store, allowing 
the store to pre-fill information based on the users description.


#### Specialisation

This approach is being promoted because any further specialisation
starts to make the database structure much more brittle. While this
is suitable for a specific type of Shop, e.g. a Music Shop, it makes
it extremely hard if they want to start selling Food but are required
by their inventory system to attribute an Album or Artist to everything.


### Metadata

Metadata can be associated with any component, with more specific
components overriding metadata from the previous.

Example:

```
  Product[GrillMaster].Manufacturer = Derp Industries
  Product[GrillMaster].Country = America
  Product[GrillMaster].Version[9000].Manufacturer = Darp Inc.
  Product[GrillMaster].Version[9000].Country = China
```

Meaning that all GrillMasters are made by Derp Industries, apart from 
the Model 9000, which is made (illicitly) by Darp Inc. in Nanjing. 


### Purchases and Downloads

Customers are able to purchase anything in the Store, their receipt in
our case would be a .torrent file. Converting between terminology a
Snatch would be an item on the customers Receipt etc.

A Purchase entails the Store authorising the customer, keeping receipt
records and handing out the appropriate .torrent files.


## Core Objects

All objects have the following fields:
```
       _id: str, Globally unique ID
      name: str, human readable name
     links: dic, objects which this one is linked to
  metadata: dic, descriptive keys/values (e.g. synopsis)
```

It is the job of the database layer to map this structure into whichever logical
storage method is used as long as the standard database operations are supported.

### Example Objects:

```javascript
{ '_id':  'a1',
  'type': 'product',
  'name': 'GrillMaster',
  'metadata': {
    'manufacturer': 'Derp Industries',
    'country': 'America',
  }
}

{ '_id':  'b2',
  'type': 'version',
  'name': '9000',
  'links': {
    'product': 'a1',
  }  
  'metadata': {
    'manufacturer': 'Darp Inc.',
    'country': 'China',
  }
}

{ '_id':  'c3',
  'type': 'variation',
  'name': 'Black, Extra Large',
  'links': {
    'product-version': ['a1','b2'],
    'product': 'a1',
    'torrent': 'D0FAA0E413695B39B12D4A4A28E1917CE9450C7B',
  },
  'metadata': {
    'comments': 'The only Extra Large model in existance',
  }
}
```

## Database Operations

Because the schema is flexible and 'standardised' regardless of the type of object the
data can be structured in a way which a small number of database operations can perform
any query necessary for an end-user.

 * Get by ID
 * Find by Name
 * Find by Link
 * Find by Metadata Search
