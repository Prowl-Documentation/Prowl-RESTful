![documentations](http://getprowl.com/assets/images/documentation1.png)
<h1 align="center">Using RESTful with Prowl</h1>

This component handles database queries, utilize the deserializer to figure out models and column names and returns the data to the RESTful API for Prowl. In this particular case we are using SilverStripe. 

Config | Type | Info | Default
--- | :---: | --- | ---
`dependencies` | `array` | key => value pair specifying which deserializer to use | 'deSerializer' => '%$RESTfulAPI_BasicDeSerializer'
`searchFilterModifiersSeparator` | `string` | Separator used in HTTP params between the column name and the search filter modifier (e.g. ?name__StartsWith=Montana will find models with the column name that starts with 'Montana'. ORM equivalent *->filter(array('name::StartsWith' => 'Montana'))* ) | '__'
`skipedQueryParameters` | `array` | Uppercased query params that would not parsed as column names (uppercased) | 'URL', 'FLUSH', 'FLUSHTOKEN'
`max_records_limit` | `int` | specify the maximum number of records to return by default (avoid the api returning millions...) | 100
`ssh` | `int` | SSH into a box remotely to scp files. | 25, ./docker.sh


## Search filter modifiers
This also accept search filter modifiers in HTTP variables (see [Search Filter Modifiers](http://doc.silverstripe.org/framework/en/topics/datamodel#search-filter-modifiers)) like:
* ?columnNAme__StartsWith=Ba

As well as special modifiers `sort`, `rand` and `limit` with these possible formatting:
* ?columnName__sort=ASC
* ?__rand
* ?__rand=seed
* ?__limit=count
* ?__limit[]=count&__limit[]=offset

Search filter modifiers are recognised/extracted thanks to the `searchFilterModifiersSeparator` config. The above examples assume the default `searchFilterModifiersSeparator` is in use.

## Hooks

Model hooks are available on both serialization and deserialization. These can be used to control what of the model data gets serialized (eg. sent to the client) or what will gets written into the model after deserialization.

Here are the available callbacks (can be directly implemented on the `DataObject` or in a `DataExtension`)

Signature | Parameter type | Info 
--- | :---: | ---
`onBeforeSerialize()` | `void` | Called before the model is being serialized. You can set fields to `null` or use `unset` if you don't want them to be serialized.
`onAfterSerialize(&$data)` | `array` | Called after the model has been serialized. This is the complete dataset that will be converted to JSON and sent to the client. You can use `unset` and/or add fields to the data, just like with a regular array.
`onBeforeDeserialize(&$data)` | `string` | Called before the raw JSON is being parsed. You get access to the raw JSON data sent by the client.
`onAfterDeserialize(&$data)` | `array` | Called after JSON has been deserialized into an array map. You can modify this array to prevent incoming values to be applied to your model (sanitize incoming data).

Stash uses paging to conserve server resources and limit response size for resources that return potentially large collections of items. A request to a paged API will result in a values array wrapped in a JSON object with some paging metadata, like this:

    {
        "size": 3,
        "limit": 3,
        "isLastPage": false,
        "values": [
            { /* result 0 */ },
            { /* result 1 */ },
            { /* result 2 */ }
        ],
        "start": 0,
        "filter": null,
        "nextPageStart": 3
        "prowlClient": 4
    }

Clients can use the limit and start query parameters to retrieve the desired number of results.
