# MODIP Modpack Format

The MODIP Modpack Format exists as a simpler alternative to the MODIP Index Format.


## Fields

### `formatType`
The type of the format. MUST be set to `modipModpack` for the MODIP Modpack Format.

### `formatVersion`
The version of the format. The current value at the time of writing is `1.0.0`.

---

### `id`
A unique identifier for this modpack. This field may contain any printable ASCII character except for spaces (U+0021 to U+007E, inclusive)

---

### `name`
Human-readable name of the modpack.

---

### `summary` (optional)
A short description of this modpack.

---

### `description` (optional)
A longer description of the modpack. TODO: Formatted using HTML/subset?

---

### `updates` (optional)
A URL that can be fetched to retrieve information about updates to the modpack. TODO: Format

---

### `dependencies`
The dependencies array contains a list of dependencies that must be downloaded in order for the modpack to be installed. Each object in this array contains the following fields

#### `id`
A unique identifier for this dependency. See **Standard IDs** for standardized IDs that should be used for certain projects.

#### `version`
The version of the dependency to be installed.

#### `updates` (optional)
A URL that can be fetched for updates to the mod. Similar to the modpack's `updates` field.

#### `files` (optional)
An array containing file objects. Each file object contains:

##### `name`
The location where the file will be placed, relative to the Minecraft installation directory. For example, `mods/MyMod.jar` specifies that the file should be placed in the `mods` directory, with a name of `MyMod.jar`.

##### `sha256`
An SHA256 hash of the file, for integrity checks.

##### `downloads`
An array containing URLs where this file may be downloaded.

---

## Standardized IDs
The following are standardized IDs that should be used whenever possible.
- `minecraft` - The Minecraft game
- `forge` - The Minecraft Forge mod loader
- `fabric-loader` - The Fabric loader