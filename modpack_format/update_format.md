# Update Format
The Update Format is a simple format that can be used to specify all available versions of a mod or modpack. It's most useful in the `updates` field in the modpack format, which allows launchers to let users update modpacks automatically.

## Fields

### `formatType`
Set to `modipUpdates` (TODO: Change name?)

---

### `formatVersion`
Set to `1.0.0`.

---

### `id`, `name`, `summary`, `description`
This fields all behave exactly like the respective fields in the modpack format.

---

### `versions`
This is an array containing version objects. Each version object is comprised of the following:

#### `name`
Human-readable name of the version.

#### `releaseDate`
The release date of the version, following ISO 8601. See the `releaseDate` field in the modpack format for more information.

#### `downloads`
An array listing URLs where the modpack zip can be downloaded.
(TODO: zip or no zip?)