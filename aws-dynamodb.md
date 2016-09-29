[🏠](README.md) › [AWS.js](aws.md) › Dynamodb

# dynamodb


## primary key

A primary key is required for each table, and can be either a single or composite key.

**Single**  
- Uses partition key (hash attribute).  
- Table is an unordered hash index.  
- Simple, fast key/value queries `WHERE item = <partition_key>`.  

**Composite Key**  
- Uses both partition (hash attribute) and sort key (range attribute).  
- Table is ordered by sort key.  
- Perform more range queries `WHERE item < <sort_key>`


## api

Most API calls will return JSON response; use a common errback function to console.log responses in development.

**describe table**  
```javascript
const params = { TableName: "Music" };
dynamodb.describeTable(params, errback);
```


## Local development

**Run dynamodb locally**  
`docker run -d -p 8000:8000 --name dynamodb peopleperhour/dynamodb`

### javascript console

The javascript console can be accessed from `http://localhost:8000/shell`.  

**Blue line bookmarklet fix**  
```javascript
javascript:(function()%7Bfunction%20addStyleString(str)%20%7Bvar%20node%20%3D%20document.createElement('style')%3Bnode.innerHTML%20%3D%20str%3Bdocument.body.appendChild(node)%3B%7DaddStyleString('a%3Anot(.btn)%2C%20%3Anot(.nav%20a)%2C%20%3Anot(.navbar%20a)%20%7B%20border-bottom%3A%20none%20!important%3B%20%7D')%7D)()```
