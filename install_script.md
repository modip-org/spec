# Install script

An install script is a fragment of data which describes how to install a mod.

Each version of a project can have a different install script.

An install script consists of a *list* of steps referencing the files in the version.

Example:
`[{"type": "placeInDirectory", "directory": "mods", "file": "ExampleMod-1.0.0.jar"}, {"type": "placeInDirectory", "directory": "coremods", "file": "ExampleMod-1.0.0-coremod.jar"}]`

## `placeInDirectory` step

Download a file into a subdirectory of the Minecraft instance directory.

Example:

`{"type": "placeInDirectory", "directory": "mods", "file": "ExampleMod-1.0.0.jar"}`

`directory` may be a nested subdirectory, e.g. `"mods/1.7.10"`. Any parent directories which don't already exist are created.
`directory` may not contain a `.` or `..` part, unless the entire value of `directory` is `"."` (which places the file in the instance directory).

## `extractZip` step

Download a file *and extract it* into a subdirectory of the Minecraft instance directory.

Example:

`{"type": "extractZip", "directory": "config", "file": "modpack-configs.zip"}`

`directory` may be a nested subdirectory, e.g. `"mods/1.7.10"`. Any parent directories which don't already exist are created.
`directory` may not contain a `.` or `..` part, unless the entire value of `directory` is `"."` (which extracts the file into the instance directory).

## `jarMod` step

Download a file and merge it into `minecraft.jar` (or `minecraft-server.jar`), overwriting files that already exist.

Note: this is considered an obsolete installation technique. Newer loaders typically add themselves to the classpath instead.

`before` and `after` allow this project's jar mod to be prioritized when multiple jar mods are present. If a mod mentioned in `before` or `after`
is not installed, this does not cause it to be installed - the entry does nothing.

Example:
`{"type": "jarMod", "file": "modloadermp-1.0.0.jar", "before": ["minecraft-forge"], "after": ["modloader"]}`

## `addClasspathLibrary` step

Download a file and add it to the classpath.

Inside the "library" field is an object identical to Minecraft's version.json library specification,
except that wherever there is a `sha1`/`size`/`url` triplet in Mojang's format, the install script
MAY instead contain a `modip_filename` which refers to one of the version's files.

(Third-party libraries should still include `sha1`/`size`/`url` to download from the Internet - they shouldn't be republished as part of the mod version)

TODO: Mojang sometimes changes this format. We should pin it down to a particular version.

Example:

```
{
    "type": "addClasspathLibrary",
    "library": {
        "name": "com.me.mylibrary:mylibrary:1.2.3",
        "downloads": {
            "artifact": {
                "modip_filename": "MyLibrary-common.jar"
            },
            "classifiers": {
                "natives-osx": {
                    "modip_filename": "MyLibrary-natives-osx.jar"
                }
            }
        },
        "natives": {
            "osx": "natives-osx"
        },
        "rules": [
            {"action": "allow", "os": {"name": "osx"}}
        ]
    },
    "before": ["com.othercorp.otherlibrary:otherlibrary"],
    "after": ["com.thirdcorp.thirdlibrary:thirdlibrary:1.2"]
}
```

`before` and `after` allow this project's classpath library to be prioritized when multiple classpath libraries are present. If a library mentioned in `before` or `after` is not installed, this does not cause it to be installed - the entry does nothing. In `before` and `after` entries, the group ID and artifact ID are mandatory, but the version is optional (if not present then it matches all versions). Versions are an exact match.

`name` must contain exactly a group ID, artifact ID, and version number.

MODIP and non-MODIP (e.g. Mojang) libraries are all considered together for sorting. i.e. you may request to install a library before one Mojang library and after another.

This example library is only required on OSX. On other platforms, this install step is a no-op.
The launcher MAY still download the OSX natives and add the library entry, with the OSX-only rule, in case the instance is later moved to a computer running OSX.

## `setMainClass` step

Updates the main class. Mainly used by loaders and standalone mods.

Example:

```
{
    "type": "setMainClass",
    "from": ["net.minecraft.client.main.Main"],
    "to": "net.fabricmc.loader.launch.knot.KnotClient"
}
```

`from` specifies what the main class should have been before. If multiple actions of this type are executed (from multiple mods), they must be linked in a chain, starting from the vanilla's main class, and then the last step in the chain indicates the actual main class. If they can't be linked in a chain, the installed mods are not compatible. `from` must be a list. The previous main class must match any value in the list.

## Client/server fork step

Uses a different installation method on the client than on the server.

Example:

```
{
    "type": "client_server_fork",
    "client_steps": [
    ],
    "server_steps": [
    ]
}
```

Note: `client_steps` and `server_steps` are always lists, and must be present even if empty.

## Framework fork step

Allows a mod to support multiple mod loaders.

```
{
    "type": "framework_fork",
    "framework_steps": {
        "<framework id>": [
        ],
        "<other framework id>": [
        ]
    },
    "preferred": "<framework id>"
}
```

Note: values in `framework_steps` are always lists, and must be present even if empty.
If no entry is present for the current framework, the mod is incompatible with it.
If the current framework is unknown (i.e. no mods are installed yet), the launcher should install the one in `preferred` (which must be one of the options).
If *multiple* supported frameworks are present (i.e. Patchwork), the launcher MAY choose a preferred one, ask the user, or give up (fall back to manual installation).

If this mechanism is used to support multiple frameworks, neither framework should be listed as a dependency, as that would require it to be installed!

Design note: Using a "fork" step instead of a conventional "if" step allows the launcher to and choose a framework if none is installed yet.
