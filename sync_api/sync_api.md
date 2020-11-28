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
	"files": [
		{
			"filename": "example_project-1.0.0.jar",
			"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; required if there are URLs
			"urls": ["https://blah/blah", "https://blah2/blah2"], // optional
			"rel": ["primary"],
			"size": 1234, // in bytes; optional
		},
		{
			"filename": "example_project-1.0.0-source.jar",
			"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; required if there are URLs
			"urls": ["https://blah/blah", "https://blah2/blah2"], // optional
			"rel": ["source"]
		},
		{
			"filename": "crazy_yet_valid_file_entry.jar"
		}
	],
	"modloader": "fabric", // WILL PROBABLY CHANGE THIS - up to the index team
	"minecraft": "1.7.10", // WILL PROBABLY CHANGE THIS - up to the index team
	"equivalent_versions": [{"id": "1.0.1-forge"}] // WILL PROBABLY CHANGE THIS - up to the index team
}
```

`rel` specifies the type of file - possibly more than one for the same file.
`"rel": "primary"` means this is the "main" download. When you click on "download project", you get this file.  
`"rel": "source"` means this is the "main source" download. When you click on "download source code", you get this file.  
All files should be displayed *somewhere*, in the order they appear in the JSON, but `rel` allows for shortcuts.  
`rel` is optional; if not specified, it's the same as an empty list.  
If there's only one item in `rel`, it can be specified as a single string (without a list).

`filename` is a default filename for the download. It SHOULD be unique within a version.
It is mandatory. Clients should ignore files without a filename.

If `sha256` is present but `urls` is not (or is empty), then the server knows about the file but isn't offering to give you a copy, perhaps for copyright reasons.
It is possible that the client can look up the SHA256 hash and find the file a different way - perhaps it already has a copy, or perhaps it can use another API which has yet to be defined.
These methods are out of scope of this document. If a client has no way to download the file, it should display the file anyway, but disable the download button.

`sha256` and `urls` could both be missing. The server knows the file exists, but doesn't actually have a copy. This should be very rare, but clients must be prepared to handle it.
Clients should display the file anyway, but disable the download button.

If `urls` are present, but `sha256` is not, clients SHOULD act as if the file is not downloadable (i.e. as if `urls` is not present).
This is for reliability reasons. Inevitably, one of the URLs will become outdated, and if a client uses it anyway, it will download the error page and try to install it into Minecraft.

`size` is completely optional; if present, clients may use it to improve the user experience, for example by populating progress bars more accurately.

`equivalent_versions` indicates different mod versions which can be substituted for this one. (SPEC TEAM TO CLARIFY)

There are no delta responses here, because version information doesn't increase in size over time. `last_updated` tells you whether the version was updated at all.

# Project description (`$BASE/project/$PROJECT/description_v1`)

Query:

`GET https://example.com/api/project/example_project_1/description_v1`

Response:

```
{
	"last_updated": "1",
	
	"display_name": "Example Project 1",
	
	// Might be displayed in search results for example.
	// If not present, you might try to derive it from the beginning of summary_one_paragraph or full_description, or just display the mod name by itself.
	"summary_one_sentence": "Test project for the MCIP sync API.",
	
	// Might be displayed in search results for example, in a system which allows more space for search results.
	// If not present, you might use summary_one_sentence, or try to derive it from the beginning of full_description.
	"summary_one_paragraph": "This is a test project for the MCIP sync API. It shows what a project description looks like when represented as JSON. This is the medium-length summary.",
	
	// Displayed on the mod's page, where there's room to display tons of information.
	// TODO: SPECIFY FORMATTING. Probably based on XML / a subset of XHTML.
	// Note that Markdown parsing is ambiguous and limited; if you want to enter Markdown, the originating server must convert it to XML.
	// For now let's assume it's plain text and newlines correspond to paragraph breaks.
	"full_description": "Pretend there are 100 lines of text here detailing every feature in the mod.",
	
	"logo": {
		"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; optional
		"urls": ["https://blah/blah", "https://blah2/blah2"] // optional
	}
	
	"screenshots": [{
		"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; optional
		"urls": ["https://blah/blah", "https://blah2/blah2"], // optional
		"caption": "A windmill milling grain" // optional
	}], // multiple screenshots allowed; display them in order
	
	"authors": [{
		"name": "Contoso Corporation", // mandatory. Spaces allowed!
		"link": "http://example.com/", // optional; go here if name is clicked on
	}], // multiple authors allowed; display them in order; when there's only space for one author, display the first one.
	
	"links": [
		{"rel": "homepage", "display_name": "Official homepage", "url": "https://example.com/example_project_1"},
		{"rel": ["wiki", "documentation"], "display_name": "Wiki", "url": "https://example.com/example_project_1_wiki"},
		{"rel": ["github", "scm", "issue_tracker"], "display_name": "GitHub", "url": "https://github.com/github/platform-samples"},
	]
}
```

Note: Unlike downloads, sha256 is optional for images, since a corrupted screenshot or logo doesn't cause much of a problem.

If a client has no way to download a logo or screenshot - because it has no `urls` and the client has no other way to fetch it - it should be treated as if it didn't exist at all. Captions shouldn't be displayed by themselves.

Similarly to download `rel`s, link `rel`s allow for consistent shortcuts. They are not required to be present. There should usually be at most one of each.
The currently standardized values are:

* `"homepage"`: Official homepage
* `"documentation"`: Reference documentation (e.g. a wiki)
* `"wiki"`: A wiki. This doesn't say what kind of information is *on* the wiki...
* `"scm"`": Source Code Management repository - e.g. GitHub. But not necessarily GitHub.
* `"issue_tracker"`": Complaints go here.

Other values MAY be used (e.g. `bitbucket`), but please use common sense so they can be standardized later. If you are unsure whether your custom value is worthy of becoming standardized, make it start with `"x-"`.

Shortcuts MAY also be based on URL; a server may show a "GitHub" shortcut based on the fact that the URL points to a GitHub project)

All links should be displayed *somewhere*, regardless of `rel`. That is what `display_name` is for.  
`rel` is optional; if not specified, it's the same as an empty list. If there's only one item in `rel`, it can be specified as a single string (without a list).

The `last_updated` value is the same for all API requests associated with the same project (e.g. `versions` and `description`). It has to be updated if the mod is updated, even if the rest of the data in the response did not change.

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