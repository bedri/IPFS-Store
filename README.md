IPFS-Store (service)
======

**IPFS-Store** aim is to provide an easy to use IPFS storage service with search capability for your project.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. 

### Prerequisites

- Java 8
- Maven 
- Docker (optional)


### Build

1. After checking out the code, navigate to the root directory
```
$ cd /path/to/ipfs-store/ipfs-store-service/
```

2. Compile, test and package the project
```
$ mvn clean package
```

3. Run the project

a. If you have a running instance of IPFS and ElasticSearch 

**Executable JAR:**

```
$ export IPFS_HOST=localhost
$ export IPFS_PORT=5001
$ export ELASTIC_CLUSTERNODES=localhost:9300
$ export ELASTIC_CLUSTERNAME=elasticsearch

$ java -jar target/ipfs-store.jar
```

**Docker:**

```
$ docker build  . -t kauri/ipfs-store:latest

$ export IPFS_HOST=localhost
$ export IPFS_PORT=5001
$ export ELASTIC_CLUSTERNODES=localhost:9300
$ export ELASTIC_CLUSTERNAME=elasticsearch

$ docker run -p 8040:8040 kauri/ipfs-store
```

b. If you prefer build all-in-one with docker-compose
```
$ docker-compose -f docker-compose.yml build
$ docker-compose -f docker-compose.yml up
```

## API Documentation



### Overview

| Operation | Description | Method | URI |
| -------- | -------- | -------- | -------- |
| store | Store content into IPFS |POST | /ipfs-store/store |
| index | Index content |POST | /ipfs-store/index |
| store_index | Store & Index content | POST | /ipfs-store/store_index |
| fetch | Get content | GET | /ipfs-store/fetch/{index}/{hash} |
| search | Search content | POST | /ipfs-store/search/{index} |
| search | Search content | GET | /ipfs-store/search/{index} |

### Details

#### Store content

Store a content (any type) in IPFS 

-   **URL:** `/ipfs-store/store`    
-   **Method:** `POST`
-   **Header:** `N/A`
-   **URL Params:** `N/A`
-   **Data Params:** 
    -   `file: [content]`

-   **Sample Request:**
```
$ curl -X POST \
    'http://localhost:8040/ipfs-store/store' \
    -F 'file=@/home/gjeanmart/hello.pdf'
```

-   **Success Response:**
    -   **Code:** 200  
        **Content:** 
```
{
    "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o"
}
```

---------------------------

#### Index content

Index IPFS content into the search engine

-   **URL** `/ipfs-store/index`
-   **Method:** `POST`
-   **Header:**  

| Key | Value | 
| -------- | -------- |
| content-type | application/json |


-   **URL Params** `N/A`
-   **Data Params**

    - `request:`
    
| Name | Type | Mandatory | Default | Description |
| -------- | -------- | -------- | -------- | -------- |
| index | String | yes |  | Index name |
| id | String | no |  | Identifier of the document in the index. id null, autogenerated |
| content_type | String | no |  | Content type (MIMETYPE) |
| hash | String | yes |  | IPFS Hash of the content |
| index_fields | Key/Value[] | no |  | Key/value map presenting IPFS content metadata|


```
{
  "index": "documents", 
  "id": "hello_doc",
  "content_type": "application/pdf",
  "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o",
  "index_fields": [
    {
      "name": "title",
      "value": "Hello Doc"
    }, 
    {
      "name": "author",
      "value": "Gregoire Jeanmart"
    }, 
    {
      "name": "votes",
      "value": 10
    }, 
    {
      "name": "date_created",
      "value": 1518700549
    }
  ]
}
```
   
-   **Sample Request:**
    
```
curl -X POST \
    'http://localhost:8040/ipfs-store/index' \
    -H 'content-type: application/json' \  
    -d '{"index":"documents","id":"hello_doc","content_type":"application/pdf","hash":"QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o","index_fields":[{"name":"title","value":"Hello Doc"},{"name":"author","value":"Gregoire Jeanmart"},{"name":"votes","value":10},{"name":"date_created","value":1518700549}]}'
``` 
   
-   **Success Response:**
    
    -   **Code:** 200  
        **Content:** 
```
{
    "index": "documents",
    "id": "hello_doc",
    "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o"
}
```

---------------------------

#### Store & Index content

Store content in IPFS and index it into the search engine

-   **URL** `/ipfs-store/store_index`
-   **Method:** `POST`
-   **Header:** `N/A`
-   **URL Params** `N/A`
-   **Data Params**

    
    -   `file: [content]`
    -   `request: `

| Name | Type | Mandatory | Default | Description |
| -------- | -------- | -------- | -------- | -------- |
| index | String | yes |  | Index name |
| id | String | no |  | Identifier of the document in the index. id null, autogenerated |
| content_type | String | no |  | Content type (MIMETYPE) |
| index_fields | Key/Value[] | no |  | Key/value map presenting IPFS content metadata|


```
{
  "index": "documents", 
  "id": "hello_doc",
  "content_type": "application/pdf",
  "index_fields": [
    {
      "name": "title",
      "value": "Hello Doc"
    }, 
    {
      "name": "author",
      "value": "Gregoire Jeanmart"
    }, 
    {
      "name": "votes",
      "value": 10
    }, 
    {
      "name": "date_created",
      "value": 1518700549
    }
  ]
}
```
   
-   **Sample Request:**
    
```
curl -X POST \
  http://localhost:8040/ipfs-store/store_index \
  -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
  -F file=@/home/gjeanmart/hello.pdf \
  -F 'request={"index":"documents","id":"hello_doc","content_type":"application/pdf","index_fields":[{"name":"title","value":"Hello Doc"},{"name":"author","value":"Gregoire Jeanmart"},{"name":"votes","value":10},{"name":"date_created","value":1518700549}]}'
``` 
   
-   **Success Response:**
    
    -   **Code:** 200  
        **Content:** 
```
{
    "index": "documents",
    "id": "hello_doc",
    "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o"
}
```

---------------------------

#### Get content

Get content on IPFS by hash

-   **URL** `http://localhost:8040/ipfs-store/fetch/{index}/{hash}`
-   **Method:** `GET`
-   **Header:**  `N/A`
-   **URL Params** `N/A`
    
-   **Sample Request:**
    
```
$ curl \
    'http://localhost:8040/ipfs-store/fetch/documents/QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o' \
    -o hello_doc.pdf 
``` 
    
-   **Success Response:**
    
    -   **Code:** 200  
        **Content:** (file)

---------------------------

#### Search contents

Search content accross an index using a dedicated query language

-   **URL** `http://localhost:8040/ipfs-store/search/{index}`
-   **Method:** `GET` or `POST` 
-   **Header:**  

| Key | Value | 
| -------- | -------- |
| content-type | application/json |

-   **URL Params** 

| Name | Type | Mandatory | Default | Description |
| -------- | -------- | -------- | -------- | -------- |
| pageNo | Int | no | 1 | Page Number |
| pageSize | Int | no | 20 | Page Size / Limit |
| sort | String | no |  | Sorting attribute |
| dir | ASC/DESC | no | ASC | Sorting direction |
| query | String | no |  | Query URL encoded (for GET call) |


-   **Data Params** 

The `search` operation allows to run a multi-criteria search against an index. The body combines a list of filters : 

| Name | Type | Description |
| -------- | -------- | -------- | 
| name | String | Index field to perform the search | 
| names | String[] | Index fields to perform the search |
| operation | See below | Operation to run against the index field | 
| value | Any | Value to compare with |



| Operation | Description |
| -------- | -------- |
| full_text | Full text search |
| equals | Equals |
| not_equals | Not equals | 
| contains | Contains the word/phrase | 
| in | in the following list | 
| gt | Greater than | 
| gte | Greater than or Equals | 
| lt | Less than  | 
| lte | Less than or Equals | 


```
{
  "query": [
    {
      "name": "title",
      "operation": "contains",
      "value": "Hello"
    },
    {
      "names": ["author", "title"],
      "operation": "full_text",
      "value": "Gregoire"
    },
    {
      "name": "votes",
      "operation": "lt",
      "value": "5"
    }
  ]
}
```

-   **Sample Request:**
    
```
curl -X POST \
    'http://localhost:8040/ipfs-store/search/documents' \
    -H 'content-type: application/json' \  
    -d '{"query":[{"name":"title","operation":"contains","value":"Hello"},{"name":"author","operation":"equals","value":"Gregoire Jeanmart"},{"name":"votes","operation":"lt","value":"5"}]}'
``` 
   
    
```
curl -X GET \
  'http://localhost:8040/ipfs-store/search/documents?page=1&size=2&query=%7B%22query%22%3A%5B%7B%22name%22%3A%22votes%22%2C%22operation%22%3A%22lt%22%2C%22value%22%3A%225%22%7D%5D%7D' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
``` 

-   **Success Response:**
    
    -   **Code:** 200  
        **Content:** 
        
```
{
  "content": [
    {
      "index": "documents",
      "id": "hello_doc",
      "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o",
      "content_type": "application/pdf",
      "index_fields": [
        {
          "name": "__content_type",
          "value": "application/pdf"
        },
        {
          "name": "__hash",
          "value": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o"
        },
        {
          "name": "title",
          "value": "Hello Doc"
        },
        {
          "name": "author",
          "value": "Gregoire Jeanmart"
        },
        {
          "name": "votes",
          "value": 10
        },
        {
          "name": "date_created",
          "value": 1518700549
        }
      ]
    }
  ]
}
],
"sort": null,
"firstPage": false,
"totalElements": 4,
"lastPage": true,
"totalPages": 1,
"numberOfElements": 4,
"size": 20,
"number": 1
}
```



## Clients

### Java

Java REST api wrapper for IPFS-Store

#### Build
```
$ cd /path/to/ipfs-store/ipfs-store-client/ipfs-store-client-java
```

```
$ mvn clean install
```


#### Dependency

```
<dependency>
    <groupId>net.consensys.tools.ipfs</groupId>
    <artifactId>ipfs-store-client-java</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>

```

#### Instanciation

Create a client

```
IPFSStoreClientService client = new IPFSStoreClientServiceImpl("http://localhost:8040");	
```

#### Interface

##### Store file

```
String store(String filePath) throws IPFSStoreClientException;
String store(InputStream file) throws IPFSStoreClientException;
```

*Example:*

```
String hash = client.store("/home/gjeanmart/hello.pdf");
```

##### Index file

```
String index(String indexName, String hash) throws IPFSStoreClientException;
String index(String indexName, String hash, String id) throws IPFSStoreClientException;
String index(String indexName, String hash, String id, String contentType) throws IPFSStoreClientException;
String index(String indexName, String hash, String id, String contentType, Map<String, Object> indexFields) throws IPFSStoreClientException;
String index(String indexName, String hash, String id, String contentType, List<IndexField> indexFields) throws IPFSStoreClientException;
```

*Example:*

```
String id = client.index("documents", "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o");
```

##### Store and Index file

```
String index(InputStream file, String indexName) throws IPFSStoreClientException;
String index(InputStream file, String indexName, String id) throws IPFSStoreClientException;
String index(InputStream file, String indexName, String id, String contentType) throws IPFSStoreClientException;
String index(InputStream file, String indexName, String id, String contentType, Map<String, Object> indexFields) throws IPFSStoreClientException;
String index(InputStream file, String indexName, String id, String contentType, List<IndexField> indexFields) throws IPFSStoreClientException;
```


*Example:*

```
String id = client.index(new FileInputStream("/home/gjeanmart/hello.pdf"), "documents", "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o", "hello_doc", "application/pdf");
```

##### get file

```
OutputStream get(String indexName, String hash) throws IPFSStoreClientException;
```

*Example:*

```
String id = client.get("documents", "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o");
```

##### search file

```
Page<Metadata> search(String indexName) throws IPFSStoreClientException;
Page<Metadata> search(String indexName, Query query) throws IPFSStoreClientException;
Page<Metadata> search(String indexName, Query query, Pageable pageable) throws IPFSStoreClientException;
Page<Metadata> search(String indexName, Query query, int pageNo, int pageSize) throws IPFSStoreClientException;
Page<Metadata> search(String indexName, Query query, int pageNo, int pageSize, String sortAttribute, Sort.Direction sortDirection) throws IPFSStoreClientException;
```

*Example:*

Search all documents

```
Page<Metadata> result = client.search("documents");
```



### Spring-Data


### CLI


### Javascript



## TODO

- Implement clients: Java, Spring-Data, CLI, Javascript, Python
- Authentication (by signature?)
- Integration tests


