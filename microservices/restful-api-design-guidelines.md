# RESTful API Design Guidelines

## Introduction
This page documents some guidelines to be used when designing and specifying RESTful APIs for microservices within the Survey Data Collection (SDC) platform.

## HATEOAS
We will not be building full [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)-complaint APIs as there is currently no clear requirement for the SDC platform that justifies the additional development complexity and effort.

## URIs
URIs should be all lowercase. The primary means for identifying a resource to access is via its UUID, as per the object identity strategy. The UUID should appear after the plural resource name without further qualification:

`GET /collectionexercises/2c82b85a-2a03-434d-8019-eae05b01c6f4`

Where a secondary means of access is supported, it must be further qualified with the name of the non-ID attribute:

`GET /parties/ref/49900000001`

Resource names in URIs should be pluralised, apart from where the identity of a secondary resource is used to further qualify the request, in which case the secondary resource name should be singular. For example, retrieving the collection exercises for a survey:

`GET /collectionexercises/survey/cb0711c3-0ac8-41d3-ae0e-567e5a1ef87`

## HTTP Status Codes
Endpoints should return the appropriate HTTP status code. Including, but not limited to:

* `HTTP 201 Created` - when a resource has been created
* `HTTP 204 No Content` - when a request for a list of resources returns an empty list
* `HTTP 400 Bad Request` - when invalid request parameters are supplied
* `HTTP 404 Not Found` - when the requested resource could not be found

## JSON Responses
Primary resource identifiers within JSON responses should use an `id` attribute. Attributes that reference secondary resource types should be named with the resource type name in camel case with an `ID` suffix. External references i.e. identifiers that have a meaning to the business and are used outside the platform (e.g. reporting unit reference, case reference etc.) should be named with a `Ref` suffix:

```json
{
  "id": "c6467711-21eb-4e78-804c-1db8392f93fb",
  "surveyID": "cb0711c3-0ac8-41d3-ae0e-567e5ea1ef87",
  "surveyRef": "221"
}
```

## JSON Schema
[JSON Schema](http://json-schema.org/) should be used for the description and validation of RESTful resources.

## Collection+JSON Hypermedia Type
In accordance with not using HATEOAS, the Collection+JSON hypermedia type (application/vnd.collection+json) should not be used. Instead simply use a JSON array where a list of objects is required.