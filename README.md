# Velmie API Specification and Requirements

---

**Version**: 1.0

---

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119].

[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt

---

## Table of Contents

1. [General](#general)
1. [Responses](#responses)
1. [Sorting](#sorting)
1. [Filtering](#filtering)
1. [Pagination](#pagination)
1. [Creating, Updating and Deleting Resources](#creating-updating-and-deleting-resources)
1. [Errors](#errors)

---

### General

Every document SHOULD contain at least **one** of the following **top-level members**:

- **data**: the document’s “primary data”

- **errors**: array or errors in the right format

- **pagination**: pagination data if it is used on that page

- **meta**: some meta object.

All Attributes must be in **camelCase**.

*Content-Type* MUST be: **application/json**.

**[⬆ back to top](#table-of-contents)**

---

### Responses

#### 200 OK

A server MUST respond to a successful request to fetch an individual resource or resource collection with a 200 OK response.

A server MUST respond to a successful request to fetch a resource collection with an array of resource object or an empty array ([]) as the response document’s primary data.

Example:

```json
{
  "data": [
    {
      "id": 1,
      "userName": "Tom",
      "age": 21
    },
    {
      "id": 2,
      "userName": "Bob",
      "age": 22
    }
  ]
}
```

A server MUST respond to a successful request to fetch an individual resource with a **resource object** or **null** provided as the response document’s primary data.

If a resource object is loaded with relationships, relationships SHOULD be embedded in the resource object. 

Relationships attribute name should be:

 - in a **plural** form (users, books ) for collection
 - in a **singular** form (group, setting) from a single resource.


Example: 

```json
{
  "data": {
    "id": 1,
    "userName": "Tom",
    "age": 22,
    "userGroup": {
      "id": 1,
      "name": "Trol"
    },
    "comments": [
      {
        "id": 1,
        "message": "cool first comment"
      },
      {
        "id": 2,
        "message": "cool second comment"
      }
    ]
  }
}
```

#### 404 Not Found

A server MUST respond with 404 Not Found when processing a request to fetch a single resource that does not exist, except when the request warrants a 200 OK response with null as the primary data (as described above).

**[⬆ back to top](#table-of-contents)**

---

### Sorting

A server MAY choose to support requests to sort resource collections according to one or more criteria (“sort fields”).

An endpoint MAY support requests to sort the primary data with a sort query parameter. The value for sort MUST represent sort fields.

``
GET /people?sort=age
``

An endpoint MAY support multiple sort fields by allowing comma-separated (U+002C COMMA, “,”) sort fields. Sort fields SHOULD be applied in the order specified.

``
GET /people?sort=age,name
``

The sort order for each sort field MUST be ascending unless it is prefixed with a minus (U+002D HYPHEN-MINUS, “-“), in which case it MUST be descending.

``
GET /articles?sort=-created,title
``

The above example should return the newest articles first. Any articles created on the same date will then be sorted by their title in ascending alphabetical order.

If the server does not support sorting as specified in the query parameter sort, it MUST return **400 Bad Request**.

**[⬆ back to top](#table-of-contents)**

---

### Filtering

To filter collection, the frontend SHOULD set the filtering value to the associated attribute name in the top query object.

``
GET /people?sort=age&username=tom&age=20
``

**[⬆ back to top](#table-of-contents)**


---


### Pagination

A server MAY choose to limit the number of resources returned in a response to a subset (“page”) of the whole set available.

To make a pagination request frontend SHOULD send query params in the next format:

- **page**: page number to get
- **limit**: items per page

In the server response should be a pagination object in the following format: 

- **currentPage**: current returned page
- **totalPages**: total pages amount
- **totlaRecords**: total record amount
- **limit**: number of items per page

Example:

```json
{
  "data": [
    {
      "id": 1
    },
    {
      "id": 2
    }
  ],
  "pagination": {
    "currentPage": 3,
    "totalPages": 10,
    "totalRecords": 92,
    "limit": 10
  }
}
```

**[⬆ back to top](#table-of-contents)**


---


### Creating, Updating and Deleting Resources

A server MAY allow resources of a given type to be created. It MAY also allow existing resources to be modified or deleted.

#### Creating Resources

A resource can be created by sending a **POST** request to a URL that represents a collection of resources. The request MUST include a single resource object as primary data. 

To create a resource with the relationship, relationships should be embedded in the resource object. 

Example: 

```json
{
  "data": {
    "userName": "Tom",
    "age": 22,
    "userGroupId": 1,
    "comments": [
      {
        "message": "cool first comment"
      },
      {
        "message": "cool second comment"
      }
    ]
  }
}
```

**[⬆ back to top](#table-of-contents)**

---


### Errors

Each error from the server should be in the following format:

- **code**: a unique code of an error. It is used to identify errors in the dictionary.
- **target**: some sort of error scope
	- **field** - the error related to a certain field
	- **common** - the error related to the whole request
- *message* (*OPTIONAL*): the error message for developers (use it only for debugging purposes)
- *source* (*OPTIONAL*): a container for additional data. Arbitrary structure:
	( **field**: resource object attribute name. Required if the target set to the field. )

Example:
	 	 	
```json
{
  "errors": [
    {
      "code": "insufficient_funds",
      "target": "common",
      "message": "Hi Nick, it seems the user has an empty balance."
    },
    {
      "code": "invalid_punctuation",
      "target": "field",
      "source": {
        "field": "userPassword"
      },
      "message": "Hi Vova, it seems that the password provided is missing a punctuation character."
    },
    {
      "code": "invalid_password_confirmation",
      "target": "field",
      "source": {
        "field": "userPassword",
        "additionalData": "bla bla bla"
      },
      "message": "Hi Lesha, it seems that the password and password confirmation fields do not match."
    }
  ]
}
```


#### Error codes

| Code | Description |
| ----------- | ----------- |
| 400 | In case of an error in the request. E.g. wrong sort options, page number, etc |
| 401 | Use it only if a user has the wrong credentials (token, login/password) |
| 403 | If a user does not have enough permission to access the requested resource |
| 422 | If there are validation errors |


**[⬆ back to top](#table-of-contents)**

---
