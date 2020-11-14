# Format Values

This document is a list of standardized values that can be used in certain fields in the format.

## Frameworks

The following list contains standardized Framework names which SHOULD be used as the respective projects ID.

- `framework-fabric-loader`
- `framework-forge`
- `framework-liteloader`
- `framework-rift`
- `framework-risugami-modloader`

These names are prefixed with `framework-` to visually differentiate them from other projects by their purpose. It also prevents possible collisions between Frameworks and other projects that have the same name (though it is encouraged to not name projects after already existing and potentially popular ones).

---

## Tags

This is a list of standardized values that can be present in the `tags` field. It is not required to use these values, although it is encouraged to use them.

- `mod`
- `modpack`
- `resourcepack`
- `texturepack`
- `datapack`
- `world`
- `framework`

---

## File Relations

Files can have the following `rel` values:

- `primary` - This is the primary file of this version, and what launchers are most likely to use
- `source` - Source code of this version
- `deobf` - Deobfuscation file for this version
- `javadoc` - Downloadable Javadoc
- `installerWin` - This is a generic installer for Windows
- `installerMacos` - This is a generic installer for macOS
- `installerLinux` - This is a generic installer for Linux
- `installerGeneric` - A generic installer
- `versionJson` - Minecraft Version JSON information, used for Frameworks
- `compressedOverrides` - This file is a modpack overrides folder, stored as a compressed file.

Generic Installers exist to allow Frameworks to be stored as Projects. If a Framework offers an installer for both Windows (e.g. .exe) and a generic jar-based installer, they should be stored as separate files with the respective `installerWin` and `installerGeneric` relations.

There are also some standard Framework-specific `rel` values:

- `forgeUniversal`
- `forgeMdk`

Non-standardized values may also be used, but they MUST be prefixed with `x-`. If you're using a non-standard value with a Framework, you SHOULD prefix it with `x-frameworkname`.

---

## Conditions

Conditions can be used to provide specific information for certain users. Below are the current standardized condition groups as well as their options.

### `environment`

Refers to the Minecraft game environment.

##### `client`

This is client-side and not a server.

#### `server`

This is server-side and not a client.

<br>

Below is an example using the `environment` group and `client` and `server` options.

```json
"allowed": {
  "environment": {
    "client": true,
    "server": false
  }
}
```

## Banned Characters
The following characters cannot be present in any fields, unless exempted.
- 0+0000 (NULL)
- U+0007 (BELL)
- U+000A (LINE FEED)
- U+0085 (NEXT LINE)

**TODO: I am bad at this. Need a better list.**