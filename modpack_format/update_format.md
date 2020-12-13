# Update Format
The Update Format is a simple format that can be used to specify all available versions of a mod or modpack. It's most useful in the `updates` field in the modpack format, which allows launchers to let users update modpacks automatically.

## Fields

### `formatType`
The type of the format. At the time of writing, it must be set to `1`.

---

### `formatVersion`
The version of the format. At the time of writing, it must be set to `1`.

---

### `versions`
This is an array containing version objects. Each version object is comprised of the following:

#### `id`
A unique identifier for this version. Launchers will use this to correlate a locally installed version of the modpack with the version in the remote versions list.

#### `name`
Human-readable name of the version.

#### `releaseDate` (optional)
The release date of the version, following ISO 8601. See the `releaseDate` field in the modpack format for more information.

#### `downloads`
An array listing URLs where the modpack zip can be downloaded.