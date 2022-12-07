# TS OData V4 Server

TS OData V4 server for node.js and Typescript

## Features

- OASIS Standard OData Version 4.0 server
- usable as a standalone server, as an Express router, as a node.js stream or as a library
- expose service document and service metadata - $metadata
- setup metadata using decorators or [metadata JSON](https://github.com/jaystack/odata-v4-service-metadata)
- supported data types are Edm primitives, complex types, navigation properties
- support create, read, update, and delete entity sets, action imports, function imports, collection and entity bound actions and functions
- support for full OData query language using [odata-v4-parser](https://github.com/jaystack/odata-v4-parser)
  - filtering entities - $filter
  - sorting - $orderby
  - paging - $skip and $top
  - projection of entities - $select
  - expanding entities - $expand
- support sync and async controller functions
- support async controller functions using Promise, async/await or ES6 generator functions
- support result streaming
- support media entities

## Controller and server functions parameter injection decorators

- @odata.key
- @odata.filter
- @odata.query
- @odata.context
- @odata.body
- @odata.result
- @odata.stream

## Example server

- server file :

```typescript
export class ProductsController extends ODataController {
  /**
   *  Get All products
   * @param query
   * @returns products
   */
  @odata.GET
  async select(@odata.query $query: ODataQuery): Promise<Product[]> {
    const db = await connect();
    const query = createQuery($query);
    const { rows } = await db.query(query.from('"Products"'), query.parameters);
    return convertResults(rows);
  }

  /**
   *  Get one product by id
   * @param key
   * @param query
   * @returns
   */
  @odata.GET
  async selectOne(
    @odata.key key: number,
    @odata.query query: ODataQuery
  ): Promise<Product> {
    const db = await connect();
    const sqlQuery = createQuery(query);
    const { rows } = await db.query(
      `SELECT ${sqlQuery.select} FROM "Products"
                                     WHERE "Id" = $${
                                       sqlQuery.parameters.length + 1
                                     } AND
                                           (${sqlQuery.where})`,
      [...sqlQuery.parameters, key]
    );
    return convertResults(rows)[0];
  }
}
```

- index file :

```typescript
import { NorthwindServer } from "./server";
require("dotenv").config();

// Start ODATA server
const port = parseInt(process.env.PORT, 10) || 3000;
export default NorthwindServer.create(port).addListener("listening", () => {
  console.log(`Odata server listening on port ${port} 🚀`);
});
```

### Response example

**Path :** `http://localhost:3003/Categories`

**Method :** `GET`

```json
{
  "@odata.context": "http://localhost:3000/$metadata#Categories",
  "value": [
    {
      "Id": 1,
      "Name": "Beverages",
      "Description": "Soft drinks"
    },
    {
      "Id": 2,
      "Name": "Grains/Cereals",
      "Description": "Breads"
    },
    {
      "Id": 3,
      "Name": "Meat/Poultry",
      "Description": "Prepared meats"
    },
    {
      "Id": 4,
      "Name": "Produce",
      "Description": "Dried fruit and bean curd"
    },
    {
      "Id": 5,
      "Name": "Seafood",
      "Description": "Seaweed and fish"
    },
    {
      "Id": 6,
      "Name": "Condiments",
      "Description": "Sweet and savory sauces"
    },
    {
      "Id": 7,
      "Name": "Dairy Products",
      "Description": "Cheeses"
    },
    {
      "Id": 8,
      "Name": "Confections",
      "Description": "Desserts"
    }
  ]
}
```
