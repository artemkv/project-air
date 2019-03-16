# REST Service language spec

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

This would create the docker container that you could pull from the Docker Hub and push into production.
The service would have 1 endpoint that:
- Returns 200 OK with movie data in the JSON response
- Returns 404 Not Found if not found
- Uses in-momory database with auto-generated table "movie", if zero config is provided
- Issues a select statement against the existing "movie" table, if connection string is provided
- Logs request/response/query


## REST Guidelines

http://addref.blogspot.com/2013/03/rest-checklist.html


## Configuration:

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
		
- Returns 200 OK with json response
- Returns 404 Not Found if not found


### Collections

```
GET "movies{?paging}" =>
	from table movie
	return many with paging:
		id,
		title,
		description,
		duration
```
		
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
		
- Returns 200 OK with json response
- Returns 400 Bad Request if pagination request is invalid


## POST

POST "movies" =>
	from body
	insert into table movie:
		title,
		description,
		duration(min=1, max=600)
		
- Takes in JSON object with properties title, description, duration
- Validates the input based on spec (see below) or directly (like with duration)
- Inserts into table "movie", with id generated
- Returns 201 Created with movie object in response body and URL in Location header


## PUT

PUT "movies/{id}" =>
	from body
	update table movie by id:
		id,
		title,
		description,
		duration(min=1, max=600)
		
- Takes in JSON object with properties id, title, description, duration
- Checks that id in route matches the one in the body
- Validates the input based on spec (see below) or directly (like with duration)
- Updates table "movie", matching record by id
- Returns 204 No Content when successful
- Returns 404 Not Found if not found


## DELETE

DELETE "movies/{id}" =>
	from table movie
	delete by id

- Returns 204 No Content when successful
- Returns 404 Not Found if not found


### Spec

movie.spec

```
int id key(identity)
string title(required, min=2, max=255, unique)
string description(required, max=1000)
int duration(required)
date createdDate(required, oninsert)
date modifiedDate(required, onupdate)
```

