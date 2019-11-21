# Storefront API Technical Vision

## Definition

*Storefront APIs* - public operations and queries that 
will be available at the application storefront for buyers.


### Desired state

![Storefront APIs](storefront-api/storefront-api-01.png)


#### Storefront APIs are separated from admin APIs.

We are using different object representation for storefront and admin scenarios.
We are applying different rules and restrictions for such objects depending on where these objects appear.
Currently, we do not have ACL resources specified for the storefront.
So there are no particular reasons to keep admin and storefront APIs together.
Make sense to separate our APIs per purposes
and extract specific modules that will serve storefront applications.

#### Storefront application and admin application are using different databases (database objects).

Actually, Magento already has this principle partially implemented.
Admin commands manipulate with raw data sources,
when queries work mostly with indexes which were pre-populated
to return data in the most efficient way.

![Storefront application and admin application](storefront-api/storefront-api-02.png)

#### Storefront queries API operate with eventually consistent data

This is a direct consequence of the previous statement.
At the moment Magento follows this principle.
Storefront queries use pre-baked data. 
So exists some delay between the moment
when data was submitted to the system
and the moment when data become available for a storefront.

#### Storefront API clients should utilize deferred execution to minimize number of requests

At the moment all storefront implementation are trees.
Blocks generates by instruction from assembled layout.
GraphQL resolving data by traversing query tree.
We can assume that all possible future storefront implementation
will stick to this approach.
This leads us to N+1 problem.
Deferring data retrieving operation can resolve it.
Technically, we do not want to have 
a few similar repeating queries during the single page load.
From the API design standpoint, this means that storefront APIs should be capable to aggregate multiple requests into one.
In the same time client should be capable to delay execution until it collects all possible request that it has to perform against the specific API. 
[More information about batch query API](https://github.com/magento/architecture/pull/163)

#### Storefront modules granularly sliced by modules

Storefront do not aggregate data into huge roots.
Actually, as usual this is responsibility of presentation.
This allows us to create small granular modules for storefront APIs.

As usual, the client performs aggregation.
For instance, [graphql](https://graphql.org) initially was designed to unify access across multiple resources.
Because we do not need to care about aggregation we can focus on forcing API to be more scalable and evolvable.

![CategoryPage](storefront-api/storefront-api-03.png)
```graphql
query {
	category {
    products {
      sku
      name
      price {
        amount
        tax {
          amount
        }
      }
    }
  }	
}

```
##### Involved APIs and modules 
```
    getProducts                     ----------> CatalogStorefront
    |
    `- getProductPrices             ----------> CatalogPriceStorefront
       |
       ` getProductPriceTaxes       ----------> CatalogTaxStorefront

```

#### Storefront modules does not depends on each other

Module purpose strictly restricted by the scenario it introduces.
The are no particular reason to do this because
all aggregation will be performed by the presentation layer.

#### Storefront APIs are replaceable

Because we are going to manage storefront APIs as a set of 
small granular modules which do not rely on each other 
we can assume that it should be pretty straightforward to replace each of them.
