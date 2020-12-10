# MODIP Modpack Format

The MODIP Modpack Format is a simple format that lets you store modpacks.

## Fields

### `formatType`
The type of the format. MUST be set to `modipModpack` for the MODIP Modpack Format.

### `formatVersion`
The version of the format, stored as a number. The current value at the time of writing is `1`.

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
A URL that can be fetched to retrieve information about updates to the modpack. This follows the update format as specified in **update_format.md**.

---

### `releaseDate`
The release date of this specific version of the modpack. It MUST be stored as an ISO-8601 conforming string. This MUST include UTC time at the end. A valid example is `2020-01-01T12:00:00Z`. This example date is the 1st of January, 2020, at 12:00:00 UTC. Spaces and time zone offsets (e.g. `+01:00`) CANNOT be used. The `Z` at the end of the string MUST be included and capitalized. The `T` separating the date and time MUST be included and capitalized.

Other values, such as `2020-W32` are NOT allowed. The only allowed format is demonstrated in the example.

---

### `files`
The files array contains a list of files for the modpack that needs to be downloaded. Each item in this array contains the following:

#### `path`
The destination path of this file, relative to the Minecraft Instance directory. For example, `mods/MyMod.jar` resolves to `.minecraft/mods/MyMod.jar`.

#### `sha256`
An SHA256 hash of the file, used for integrity checks.

#### `downloads`
An array containing URLs where this file may be downloaded.

---

### `dependencies`
This object contains a list of IDs and version numbers that launchers will use in order to know what to install.

Available dependency IDs are:
- `minecraft` - The Minecraft game
- `forge` - The Minecraft Forge mod loader
- `fabric-loader` - The Fabric loader

An example `dependencies` object:
```json
"dependencies": {
    "minecraft": "1.16.4",
    "fabric-loader": "0.10.8"
}
```

---

## Storage
When stored on disk, the modpack MUST be in ZIP format, using the `.zip` extension. The main metadata of the modpack MUST be stored at `index.modip.json` in the root of the zip.

The zip may also contain a directory named `overrides`. Files in this directory will be copied to the root of the Minecraft Instance directory upon installation by the launcher. For example:
```
my_modpack.zip/
    index.modip.json
    overrides/
        config/
            mymod.cfg
        options.txt
```
When installed, the contents of `overrides` will be copied to the Minecraft Instance directory and end up similar to this:
```
.minecraft/
    config/
        mymod.cfg
    options.txt
```