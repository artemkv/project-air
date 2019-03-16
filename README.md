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


## Configuration:

- hostname "localhost"
- port "8080"
- api root "/api/"
- max page size


## GET APIs

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

