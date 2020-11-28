# Sync API

The sync API allows one server (acting as a client in this case) to download all the data from another server and keep its copy up-to-date.

It's a hierarchical API, so the client can skip data blobs it's not interested in or doesn't support.

The base URL in the examples is `https://example.com/api/`

# Top level - project list (`$BASE/project_list_v1`)

Query:

`GET https://example.com/api/project_list_v1`

Result:

```
{
	"last_updated": "1",
	"projects": [
		{"id": "example_project", "uuid": "12345678-1234-1234-1234-12345678abcd", "last_updated": {"versions": "1", "description": "1"}},
		{"id": "example_project_2", "uuid": "12345678-1234-1234-1234-12345678abce", "last_updated": {"versions": "1", "description": "1"}},
		{"id": "example_repo_1:example_imported_project", "uuid": "12345678-1234-1234-1234-12345678abcf", "last_updated": {"versions": "1", "description": "1"}},
		{"id": "example_repo_1:example_repo_2:example_repo_3:example_nested_imported_project", "uuid": "12345678-1234-1234-1234-12345678abd0", "last_updated": {"versions": "1", "description": "1"}}
	],
	
	// optional
	"suggested_polling_rate": 300,
}
```

The client can save `last_updated` in order to receive a delta next time it polls.

The primary identifier of a project is the ID, not the UUID.

The project UUID is used to avoid cycles. If the client already has the same UUID via a shorter route, then it won't download that project from this server.
Servers which use the same UUID for different projects, or copy the UUID of existing projects, are considered malicious (or just broken).

The UUID must be a UUID (32 hexadecimal characters, 3 hyphens, 8-4-4-4-12 arrangement) and **must be in lowercase**.

## Delta fetch

Query:

`GET https://example.com/api/project_list_v1?last_updated=1`

Result:

```
{
	"last_updated": "2",
	"projects": [
		{"id": "example_project", "uuid": "12345678-1234-1234-1234-12345678abcd", "last_updated": {"versions": "76", "description": "58"}},
		{"id": "example_project_2", "uuid": "12345678-1234-1234-1234-12345678abce", "last_updated": {"versions": "0", "description": "485bc531-c25f-476d-a6e6-4d6cec094d11"}},
		{"id": "example_repo_1:example_imported_project_2", "uuid": "12345678-1234-1234-1234-12345678abd1", "last_updated": {"versions": "flubber", "description": "i'm a potato"}},
	],
	"deleted_projects": [
		{"id": "example_repo_1:example_repo_2:example_repo_3:example_nested_imported_project"}
	],
	
	// optional
	"suggested_polling_rate": 300,
}
```

This response only contains projects which were created, updated, or deleted, since the last request.
The `last_updated` field from the previous response is used as a parameter in the request.

`last_updated` values don't have to be timestamps or even numbers at all. However, they must always be represented in JSON as strings.
For example, they could be timestamps, sequence numbers, or git hashes. Servers may even mix-and-match, as seen above.
Clients shouldn't try to compare `last_updated` values.
The only rule is: if the value is the same as last time, then the object hasn't changed.

Note that multiple API objects are associated with a project. Each file has its own `last_updated` value.
It is meaningless to compare `last_updated` values for different objects.

The presence of `deleted_projects` indicates that the response is a delta response.
The server can always choose to send a full response, in which case the `deleted_projects` key is not present.
In a full response, any project which is not present in `projects` has been deleted.

Note that the server must store all project names that were ever deleted, in order to generate delta responses.
The server could delete this history, but then it will need to record the fact that it deleted the history, and it cannot send delta responses from earlier `last_updated` values.
Whenever clients make requests with `last_updated`, expecting delta responses, they MUST be prepared to accept full responses instead.

## Polling rate

Clients which poll should avoid using a fast polling rate if they are not receiving incremental responses.

Polling rate is ultimately up to the client administrator. The recommended defaults and minimum values are the lowest of:

* If the previous two requests for this object resulted in delta responses, or less than two requests have been made so far:
    * Minimum: every 5 minutes
    * Default: every 20 minutes
* Otherwise:
    * Minimum: every 2 hours
    * Default: every 12 hours
* If the previous response included a `suggested_polling_rate` key:
    * Minimum: One third of `suggested_polling_rate` seconds
    * Default: Every `suggested_polling_rate` seconds

The non-delta default rate is quite long, because the project list could be quite large.
If a server only hosts a few projects, its administrator might not mind fast polling even if the server doesn't support delta responses.
In this case, the administrator should set `suggested_polling_rate` to a low value.

# Project version list (`$BASE/project/$PROJECT/versions_v1`)

Query:

`GET https://example.com/api/project/example_project/versions_v1`

Response:

```
{
	"last_updated": "1",
	"versions": [
		{"id": "1.0.0", "last_updated": "1"},
		{"id": "1.0.0-interim-fix", "last_updated": "1"}
		{"id": "1.0.1", "last_updated": "1"},
	]
}
```


## Delta

Query:

`GET https://example.com/api/project/example_project/versions_v1?last_updated=1`

Response:

```
{
	"last_updated": "76",
	"versions": [
		{"id": "1.0.1-fabric", "last_updated": {"main": "1"}},
		{"id": "1.0.1-forge", "last_updated": {"main": "1"}}
	],
	"deleted_versions": [
		{"id": "1.0.0-interim-fix"}
	]
}
```

Note: one might think that versions are immutable. In practice, sometimes they are not. That decision is up to the originating server. For example, a version may have been initially released with incorrect dependency information, which is later corrected. Or a warning label may be added to a version, to indicate that it's known to cause world corruption. Either one of these counts as a version update. Versions which don't load at all might even be deleted.

Versions don't have UUIDs, because there's no need to combine information from different servers.
That is to say: all of the information about a particular project originates from the same source, so it only needs to be tracked for the project as a whole.

# Project version information (`$BASE/project/$PROJECT/version/$VERSION/v1`)

Query:

`GET https://example.com/api/project/example_project/version/1.0.1-fabric/v1`

Response:

```
{
	"last_updated": "1",
	// all other data according to main spec
}
```

This is a version object as defined in the main spec, but with the addition of `last_updated`. The details are out-of-scope here.

There are no delta responses here, because version information doesn't increase in size over time. `last_updated` tells you whether the version was updated at all.

# Project description (`$BASE/project/$PROJECT/description_v1`)

Query:

`GET https://example.com/api/project/example_project_1/description_v1`

Response:

```
{
	"last_updated": "1",
	// all other data according to main spec, except for versions
}
```

This is a project object as defined in the main spec, but with the addition of `last_updated`, and there is no `versions`. The details are out-of-scope here.

There are no delta responses here, because this information doesn't increase in size over time. `last_updated` tells you whether the description was updated at all.

# Limits

We can't stop you from publishing whatever you want, so this section defines a minimum capability for consumers, rather than a maximum capability for publishers.
Publishers would do well to obey them in order to avoid consumers of their content from encountering problems.

* Each `last_updated` should be no longer than 128 characters. If this limit is violated, clients may be unable to use the delta update mechanism, and may fall back to full updates.
* UUIDs are exactly 35 characters long. They can be unambiguously converted to a 16-byte binary representation for storage.
* Version IDs should be no longer than 128 characters. If this limit is violated, clients MAY be unable to show the versions with longer IDs.

Limits on the contents of a version are outside the scope of this document.

## Project IDs

Because of the prefix system, there is no guaranteed maximum length for project IDs. Clients MAY choose to ignore projects with IDs longer than 255 characters.

Projects should not be originated with IDs longer than 63 characters. This allows for up to 12 15-character prefixes with separators, before the certain clients may not be able to handle the project. (There is no limit on number or length of prefixes, so this is an approximate limit)

Note that if the length limit is reached and causes a problem, an effective workaround is to create a shortcut along the path - i.e. if `server_a:server_b:server_c:...:server_z:project_id` is too long, `server_a` may choose to directly trust `server_z`, thus shortening the ID to `server_a:server_z:project_id`

# other stuff

TODO: What else is needed?