[🏠](README.md) › [AWS.js](aws.md) › Dynamodb

# dynamodb

## Naming conventions

**Tables** Collection of items. Equivalent of postgres table
**Item**  Collection of attributes. Equivalent of a postgres `row` or mongodb `document`.  
**Attribute** key:value pairs. Equivalent of a postgres `column` or mongodb `field`.  


## Tables
Schemaless, except for the attribute(s) that make up a primary key need to be on each item.


## primary key

A primary key is required for each table, and can be either a single or composite key. The primary key has to be unique (can have duplicate hash attributes for composite keys).  

**Single**  
- Uses partition key (hash attribute).  
- Table is an unordered hash index.  
- Simple, fast key/value queries `WHERE item = <partition_key>`.  

**Composite Key**  
- Uses both partition (hash attribute) and sort key (range attribute).  
- Table is ordered by sort key.  
- Perform more range queries `WHERE item < <sort_key>`


## Global Secondary Indexes
- Similar to tables in that you can configure write and read i/o.

## api
Examples for interacting with the dynamodb api using the Node.js sdk and aws cli. For node.js use a common errback function to console.log responses in development.  


### errback  

```javascript  
const logJson = data => console.log(JSON.stringify(data, null, 2));
const errback = (err, data) => (err) ? logJson(err) : logJson(data);
```

### describe table  

**node.js**  
```javascript
const params = { TableName: "Music" };
dynamodb.describeTable(params, errback);
```

### list table  

**node.js**  
```javascript
const params = {};
dynamodb.listTables(params, errback);
```

**cli**  
`aws dynamodb list-tables`


### conditional insert  
Same as traditional insert but we add the ConditionalExpression field.  
On success returns empty map: `{}`.

**node.js**  
```javascript
const params = {
  TableName: "sl_12355543664",
  Item: {
    "Artist": "Damien Rice",
    "SongTitle": "9 Crimes",
  },
  "ConditionExpression": "attribute_not_exists(Artist) and attribute_not_exists(SongTitle)"
};
docClient.put(params, errback);
```

### Batch insert  
On success, returns `{ "UnprocessedItems": {} }`

**node.js**  
```javascript
const Music = [{
  PutRequest: { Item: {
    "Artist": "Damien Rice",
    "SongTitle": "9 Crimes",
  }}}, {
  PutRequest: { Item: {
    "Artist": "Damien Rice",
    "SongTitle": "The Blower's Daughter",
    "AlbumTitle":"O",
  }}},
];
const params = { RequestItems: { Music } };
docClient.batchWrite(params, errback);
```

**cli**  
forum.json is assumed to be in cwd.  
`aws dynamodb batch-write-item --request-items file://forum.json`


### Read item

**cli**  

Passing JSON inline
`aws dynamodb get-item --table-name Forum --key '{"Name": {"S": "Amazon S3"}}'`

Indexes

Two types: Global Secondary Index (GSI) Local Secondary Index (LSI)

- Secondary indexes can include projected attributes, so common queries and their attributes can be indexed. SEE: [Creating the DynamoDB Table](https://aws.amazon.com/blogs/compute/using-amazon-api-gateway-as-a-proxy-for-dynamodb/#Creating%20the%20DynamoDB%20Table)

**projection expression**  
A projection expression is a string that identifies the attributes you want.

**expression attribute value**  
If you need to compare an attribute with a value, define an expression attribute value as a placeholder. Expression attribute values are substitutes for the actual values that you want to compare — values that you might not know until runtime.

**Scan**  
Get all items with scan


### Delete table

**cli**  
`aws dynamodb delete-table --table-name <table_name>`



## Local development

**Run dynamodb locally**  
`docker run -d -p 8000:8000 --name dynamodb peopleperhour/dynamodb`

### javascript console

The javascript console can be accessed from `http://localhost:8000/shell`.  

**Blue line bookmarklet fix**  
```javascript
javascript:(function()%7Bfunction%20addStyleString(str)%20%7Bvar%20node%20%3D%20document.createElement('style')%3Bnode.innerHTML%20%3D%20str%3Bdocument.body.appendChild(node)%3B%7DaddStyleString('a%3Anot(.btn)%2C%20%3Anot(.nav%20a)%2C%20%3Anot(.navbar%20a)%20%7B%20border-bottom%3A%20none%20!important%3B%20%7D')%7D)()```

**further reading**  

[AWS Service Proxy](https://aws.amazon.com/blogs/compute/using-amazon-api-gateway-as-a-proxy-for-dynamodb/)
