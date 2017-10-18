# Microservice Object Identifer Strategy

## Introduction
This page documents the strategy to be used for uniquely identifying objects exposed by the microservices comprising the Survey Data Collection platform.

## Design Principles
The proposed solution must adhere to the following principles:

* Don’t leak implementation details
* Once assigned to an object the identifier must be immutable and stay with the object forever
* The identifier should be sufficient to uniquely identify an object and verify if two instances are the same object

## Approach
Objects passed between microservices, either via a RESTful API or a queue, will be assigned a [Universally Unique IDentifier (UUID)](https://en.wikipedia.org/wiki/Universally_unique_identifier) exposed to other microservices within an `id` field contained in the response JSON or queue message.

The use of UUIDs as object identifiers avoids the problem of surfacing predictable IDs, which in particular can cause problems within user interfaces by exposing a count of records, or prompting an attacker to try substituting a different integer ID within a URL in an attempt to retrieve someone else's data. This second issue should always be mitigated by an authorisation check within the application code, but it's even better not to invite it in the first place.

The following rules must be adhered to during UUID generation:

* UUIDs must be generated using the version 4 (random) algorithm
* UUIDs must be generated within the application code (examples are given below), not within the database. This avoids the problem whereby every time you need to insert an entity, you need the database to give you its primary key i.e. the entities in your code are incomplete until you talk to the database
* UUIDs must be stored in lowercase using the 8-4-4-4-12 format e.g: `123e4567-e89b-12d3-a456-426655440000`
* See [RFC 4122](https://tools.ietf.org/html/rfc4122) for full details of UUIDs

## URNs
The use of [Uniform Resource Names](https://en.wikipedia.org/wiki/Uniform_Resource_Name) (URNs) was considered, but rejected. URNs allow the identity of a RESTful resource (object) to be separated from its location. A good analogy is to think of how we generally refer to people by their name rather than their home address, because the latter can change. Using URNs requires the use of a URN resolver, which is like an address book that looks up the URN it is given and produces the [Uniform Resource Identifier](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) (URI) where that resource can be found. The use of a URN resolver increases network traffic because every reference to a resource requires an additional call to look up its location.

There is currently no clear requirement for the SDC platform that justifies the additional development and operational complexity that would be required to fully implement URNs. If a requirement emerges later it would be easy to add to the strategy proposed here by prefixing all existing UUIDs with a URN namespace identifier (NID) such as `uk.gov.ons.<type>`

## Worked Example
The example below illustrate how the use of UUIDs to identify objects would work in practice. Rather than get bogged down in the detail the important thing is to note how the UUIDs are passed around between the three collaborating microservices.

For this example the Collection Instrument service needs to load an offline collection instrument (spreadsheet) into the system so it's associated with the correct reporting unit. The RAS back office UI makes a request to the Survey service to get the list of known surveys so it can present their names to the user:

`GET https://surveysvc/surveys`

```json 
[{
    "id": "cb0711c3-0ac8-41d3-ae0e-567e5ea1ef87",
    "shortName": "BRES"
}]
```

The user selects the survey applicable to the collection instrument they want to load e.g. BRES. At this point the application has the unique identifier for the BRES survey object and can ask the Collection Exercise service for the list of collection exercises for that survey:

`GET https://collectionexercisesvc/collectionexercises/survey/cb0711c3-0ac8-41d3-ae0e-567e5ea1ef87`

```json
[{
    "id": "2c82b85a-2a03-434d-8019-eae05b01c6f4",
    "name": "201601",
    "scheduledExecutionDateTime": "2017-05-10 11:55:00Z"
},
{
    "id": "...",
    "name": "...",
    "scheduledExecutionDateTime": "..."
}]
```

The collection exercise details are displayed to the user, who selects the relevant collection exercise. The Collection Instrument service can now persist the spreadsheet by making an HTTP PUT request to update the relevant collection instrument resource. Note how the request payload contains the UUID for the collection exercise retrieved in the previous step:

`PUT https://collectioninstrumentsvc/collectioninstruments/6A33A6C9-F01F-4224-81D6-362C211DFD31`

```json
{
    "classifiers": [{
        "classifierType": "RU_REF",
        "classifier": "49900000001"
    },
    {
        "classifierType": "COLLECTION_EXERCISE",
        "classifier": "2c82b85a-2a03-434d-8019- eae05b01c6f4"
    }]
}
```

# Disadvantages
UUIDs are not a silver bullet. There are some disadvantages inherent with this approach:

* Uniqueness is only enforced within an individual microservice's database by a constraint, not globally across the platform. There exists the very, very remote possibility of an identifier collision across the services within the platform. This risk may be increased slightly by a polyglot approach to microservice development with different programming languages using different library implementations of the v4 UUID generation algorithm
* Each UUID requires 16 bytes of storage
* UUIDs are unwieldy to work with

# Code Examples
## PostgreSQL
PostgreSQL contains a native `uuid` data type:

```sql
CREATE TABLE "example" (
    id uuid NOT NULL
);
```

```sql
ALTER TABLE "example" ADD CONSTRAINT example_id_key UNIQUE (id);
```

UUIDs can be inserted into tables as strings:

```sql
INSERT INTO example(id) VALUES ('123e4567-e89b-12d3-a456-426655440000');
```

## Programming Languages
The examples below show how to create string representations of v4 UUIDs in four different programming languages.

### Go
`go get github.com/satori/go.uuid`

```go
import (
        "fmt"
        "github.com/satori/go.uuid"
)

fmt.Printf("%s\n, uuid.NewV4())
```

### Java
```java
import java.util.UUID;
 
String.valueOf(UUID.randomUUID())
```

### Python
```python
import uuid
 
str(uuid.uuid4())
```

### Ruby
```ruby
require 'securerandom'
 
SecureRandom.uuid
```