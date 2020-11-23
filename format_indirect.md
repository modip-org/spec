# Indirect Format

The Indirect Format allows for a project to be referenced in a smaller format that takes up less space and bandwidth. 

An indirect project always starts with the `"indirect": true` value in order to clearly indicate that it's indirect. This allows launchers and clients to understand and treat this differently.

At the root of the format are many of the same fields that are in the root of the regular format. These are `schemaVersion`, `id`, `name`, `summary`, `media`, `rel`, `authors`, `license`, `links`, `successors`, and `predecessors`. This fields act exactly the same as they do in the full format, so read **format_spec.md** to find out more.

The indirect format contains modified versions of original fields.

### `description`

This is the field for the full description. It contains a URL that, when fetched, responds with a document which follows the Formatted Text standard. For example, the project is listed like this:
```jsonc
{
  "id": "my-project",
  "name": "My Project",
  "description": "https://example.com/db/example-project/description.xml"
}
```

When you make a HTTP GET request to `https://example.com/db/example-project/description.xml`, you get a result similar to:
```xml
<h1>My Project</h1>
<h3>This is my Project</h3>

My project has many cool features.
```

### `versions`

This is the field for specifying versions. It still contains objects with information, but the amount of information is cut down.

Each object stored contains some of the same fields stored in the regular, full-size object. These fields are: `id`, `name`, `semver`, and `releaseDate`.

If you want to retrieve other information, like the list of files or installation scripts, you'll need to make a separate request to a URL specified in a separate field, `full`.

`full` is a URL linking to full metadata of the version. This metadata starts directly at the version object level. It also includes the `schemaVersion` object, to make sure versions are still comparable correctly. Below is an example:

```jsonc
// GET https://example.com/db/example-project/1.0.0-full.modip.json returns:
{
  "schemaVersion": "1.0.0",
  "id": "my-version",
  "name": "My Version Name",
  "semver": "1.0.0",
  "releaseDate": "2020-10-30T12:00:00Z",
  "installation": [
    /** stuff goes here **/
  ],
  "files": [
    {
      "name": "my-file.jar"
      /** you get the idea **/
    }
  ]
}
```

The `schemaVersion` field MUST exist, otherwise the data will be invalid and backwards-compatibility cannot be properly implemented.

// TODO: More more fields behind other requests?


# Format Example

```jsonc
{
  "schemaVersion": "1.0.0",
  "id": "example-project",
  "name": "Example Project",
  "description": "https://example.com/db/example-project/description.xml",
  "versions": [
    {
      "id": "my-version",
      "name": "My Version Name",
      "semver": "1.0.0",
      "releaseDate": "2020-10-30T12:00:00Z",
      "full": "https://example.com/db/example-project/1.0.0-full.modip.json"
    }
  ]
}
```
