# REST Service Language Spec

Just imagine you could write 7 lines of code like this to get a complete service that you could immediately deploy in the cloud:

```
GET "movies/{id}" =>
	from table movie
	return by id:
		id,
		title,
		description,
		duration
```

This would create the docker container that you could pull from the DockerHub and push into production.
The service would have 1 endpoint that:
- Returns 200 OK with movie data in the JSON response
- Returns 404 Not Found if not found
- Uses in-momory database with auto-generated table "movie", if zero config is provided
- Issues a select statement against the existing "movie" table, if connection string is provided
- Logs request/response/query
- etc


## REST Guidelines

http://addref.blogspot.com/2013/03/rest-checklist.html


## Optional Configuration

api.config

```
- server.address:0.0.0.0
- server.port:8080
- api.root:api
- api.paging.maxPageSize:100
- api.allowOrigins=*
```


## GET

### Single object

```
GET "movies/{id}" =>
	from table movie
	return by id:
		id,
		title,
		description,
		duration
```
		
- Returns 200 OK with JSON response
- Returns 404 Not Found if not found


### Collections

Paging:

```
GET "movies{?paging}" =>
	from table movie
	return many with paging:
		id,
		title,
		description,
		duration
```

Paging + filter:
		
```
GET "movies{?paging}&{?filter}" =>
	allow filter on: exact(title), contains(description), lessThan(duration)

	from table movie
	return many with paging, filter:
		id,
		title,
		description,
		duration
```

Paging + sorting:

```
GET "movies{?paging}&{?sorting}" =>
	allow sorting on: id, title, duration

	from table movie
	return many with paging, sorting:
		id,
		title,
		description,
		duration
```

Paging + sorting + filter:

```
GET "movies{?paging}&{?sorting}&{?filter}" =>
	allow sorting on: id, title, duration
	allow filter on: exact(title), contains(description), lessThan(duration)

	from table movie
	return many with paging, sorting, filter:
		id,
		title,
		description,
		duration
```

Extra filter not coming from request:

```
GET "movies{?paging}&{?sorting}&{?filter}" =>
	allow sorting on: id, title, duration
	allow filter on: exact(title), contains(description), lessThan(duration)

	from table movie
	return many with paging, sorting, filter:
		id,
		title,
		description,
		duration
	where
		deleted = false
```
		
- Returns 200 OK with JSON response
- Returns 400 Bad Request if pagination request is invalid


## POST

```
POST "movies" =>
	from body
	insert into table movie:
		title,
		description,
		duration(min=1, max=600)
```
		
- Takes in JSON object with properties title, description, duration
- Validates the input based on spec (see below) or directly (like with duration)
- Inserts into table "movie", with id generated
- Returns 201 Created with movie object in response body and URL in Location header


## PUT

```
PUT "movies/{id}" =>
	from body
	update table movie by id:
		id,
		title,
		description,
		duration(min=1, max=600)
```
		
- Takes in JSON object with properties id, title, description, duration
- Checks that id in route matches the one in the body
- Validates the input based on spec (see below) or directly (like with duration)
- Updates table "movie", matching record by id
- Returns 204 No Content when successful
- Returns 404 Not Found if not found


## DELETE

```
DELETE "movies/{id}" =>
	from table movie
	delete by id
```

- Returns 204 No Content when successful
- Returns 404 Not Found if not found


## Spec

movie.spec

```
int id key(identity)
string title(required, min=2, max=255, unique)
string description(required, max=1000)
int duration(required)
date createdDate(required, oninsert)
date modifiedDate(required, onupdate)
```


## Extra logic

### Calculated properties

```
using System;
using Some.Code.Calculating.Rating;
using Some.Logger;

// Dependencies are injected
IRatingCalculator _calculator;
ILogger _logger;

GET "movies/{id}" =>
{
	// Logic before request
	_logger.Log(context.Request.Url);
}
	from table movie
	return by id:
		id,
		title,
		description => description.Substring(0, 100),
		duration,
		someConstant = 5,
		rating = {_calculator.getRating(id)}
{
	// Logic after request
	_logger.Log(context.TimeElapsed);
}
```

- Fields with lambda allow extra processing on database fields
- Fiels returned with "field =" syntax are not mapped to db and always calculated
- Implicit "context" variable is always defined that contains some extra info