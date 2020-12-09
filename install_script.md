# Install script

An install script is a fragment of data which describes how to install a project.

Each version of a project can have a different install script.

An install script consists of a *list* of steps referencing the files in the version.

Example:
```json
[
    {
        "type": "placeInDirectory", 
        "directory": "mods", 
        "file": "ExampleMod-1.0.0.jar"
    }, 
    {
        "type": "placeInDirectory", 
        "directory": "coremods", 
        "file": "ExampleMod-1.0.0-coremod.jar"
    }
]
```

## `depends`/`conflicts`/`recommends` step

`depends` means the referenced project is required for this one to work correctly.  
`conflicts` means this project does not work alongside the referenced one.  
`recommends` means this project interoperates with the referenced one. This is not binding.

Attempting to install a project without a dependency should install the dependency, or prompt the user to install it.  
Attempting to install a project that conflicts with an already-installed one should prompt the user to uninstall it.  
Expert users should have the option to ignore both problems, and install the project anyway.

Finding a set of dependencies which do not conflict is, in the general case, equivalent to solving the Boolean satisfiability problem ("SAT").
Installers are not required to include SAT solvers, since dependencies and conflicts are advisory. If the user wants to install a set of projects, and an "obvious" solution does not work (e.g. latest version of each), then the installer may leave it up to the user to find a set of versions which works, instead of spending significant computational time trying to find a solution. Installers are permitted to include additional heuristics.

Example:

```json
[
    {
        "type": "depends", 
        "id": "cofh-core", 
        "version": ">=1.2.3.4", 
        "url": "https://example.com/modip/cofh-core.json"
    }
]
```

SpecialID example:

```json
[
    {
        "type": "depends", 
        "specialID": "minecraft", 
        "version": "1.6.4"
    }
]
```

### `id`

The ID of the required dependency.

### `url`

URL pointing to the file with information about this dependency?  
Only present when `id` is present (not `specialID`)  
[TODO: is this actually needed? Or should you query the server using the ID?]

### `version`

This field MUST be either an Array or String. If this field is an array, it MUST contain a list of compatible versions. If this field is a String, it MUST contain a Semantic Versioning comparison String. An example of a comparison string is `>= 25.0.219`.
[TODO: since mod versions, Minecraft versions and Java versions don't have to follow semver, replace this definition]

### `specialID`

This can be present instead of `ID` for a special-case dependency.

Currently defined IDs:

* `minecraft` - specifies a dependency on Minecraft (Java edition)
* `java` - specifies a dependency on Java. Java is always required to run Minecraft, but this allows mods to require or conflict with specific Java versions.

## `placeInDirectory` step

Download a file into a subdirectory of the Minecraft instance directory.

Example:

```json
{
    "type": "placeInDirectory", 
    "directory": "mods",
    "file": "ExampleMod-1.0.0.jar"
}
```

`directory` may be a nested subdirectory, e.g. `"mods/1.7.10"`. Any parent directories which don't already exist are created.
`directory` may not contain a `.` or `..` part, unless the entire value of `directory` is `"."` (which places the file in the instance directory).

## `extractZip` step

Download a file *and extract it* into a subdirectory of the Minecraft instance directory.

Example:

```json
{
    "type": "extractZip", 
    "directory": "config", 
    "file": "modpack-configs.zip"
}
```

`directory` may be a nested subdirectory, e.g. `"mods/1.7.10"`. Any parent directories which don't already exist are created.
`directory` may not contain a `.` or `..` part, unless the entire value of `directory` is `"."` (which extracts the file into the instance directory).

## `jarMod` step

Download a file and merge it into `minecraft.jar` (or `minecraft-server.jar`), overwriting files that already exist.

Note: this is considered an obsolete installation technique. Newer loaders typically add themselves to the classpath instead.

`before` and `after` allow this project's jar mod to be prioritized when multiple jar mods are present. If a mod mentioned in `before` or `after`
is not installed, this does not cause it to be installed - the entry does nothing.

Example:
```json
{
    "type": "jarMod", 
    "file": "modloadermp-1.0.0.jar", 
    "before": ["minecraft-forge"], 
    "after": ["modloader"]
}
```

## `addClasspathLibrary` step

Download a file and add it to the classpath.

Inside the "library" field is an object identical to Minecraft's version.json library specification,
except that wherever there is a `sha1`/`size`/`url` triplet in Mojang's format, the install script
MAY instead contain a `modipFilename` which refers to one of the version's files. (If the installer generates a file for Minecraft's launcher,
then it must fill in the `sha1`/`size`/`url` based on the named file)

(Third-party libraries should still include `sha1`/`size`/`url` to download from the Internet - they shouldn't be republished as part of the mod version)

TODO: Mojang sometimes changes this format. We should pin it down to a particular version.

Example:

```json
{
    "type": "addClasspathLibrary",
    "library": {
        "name": "com.me.mylibrary:mylibrary:1.2.3",
        "downloads": {
            "artifact": {
                "modipFilename": "MyLibrary-common.jar"
            },
            "classifiers": {
                "natives-osx": {
                    "modipFilename": "MyLibrary-natives-osx.jar"
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

```json
{
    "type": "setMainClass",
    "from": ["net.minecraft.client.main.Main"],
    "to": "net.fabricmc.loader.launch.knot.KnotClient"
}
```

`from` specifies what the main class should have been before. If multiple actions of this type are executed (from multiple mods), they must be linked in a chain, starting from the vanilla's main class, and then the last step in the chain indicates the actual main class. If they can't be linked in a chain, the installed mods are not compatible. `from` must be a list. The previous main class must match any value in the list.

It may be assumed that the chain does not contain loops.

## Client/server fork step

Uses a different installation method on the client than on the server.

Example:

```json
{
    "type": "clientServerFork",
    "clientSteps": [
    ],
    "serverSteps": [
    ]
}
```

Note: `clientSteps` and `serverSteps` are always lists, and must be present even if empty.

## Generic fork step

Allows a mod to support multiple dependency sets, such as different mod loaders.

```json
{
    "type": "fork",
    "id": "<fork id>",
    "branches": {
        "<id 1>": {
            "label": "<label 1>",
            "steps": [
            ]
        },
        "<id 2>": {
            "label": "<label 2>",
            "steps": [
            ]
        }
    }
}
```

Note: One entry *must* be chosen. If you want an empty option, you must write it explicitly.

The installer should ask the user which branch to take. If exactly one branch is valid for the current installation (based on dependency and conflict information), the installer may skip the question.

When the version of a project is changed, the IDs may be used to (with best effort) transfer the previous fork selection to the new version.
Fork IDs are scoped to a project - not only a version. Branch IDs are scoped to a fork - different forks may use the same branch IDs, with no consequence.

Any particular "execution" of the script must not encounter two forks with the same ID. If this occurs, the behaviour is unspecified. Fork IDs may be reused on disjoint code paths - for example, in both branches of an outer fork.

One branch must be taken. It is not valid for an installer to take no branch. If all branches are invalid because of dependencies or conflicts, the mod can't be installed without overriding them. If it is valid to not take a branch, an explicit empty branch must be included. Note that even an empty branch must still contain `"steps": []`

Design note: Using a "fork" step instead of a conventional "if" step allows the launcher to enumerate all valid configurations.
