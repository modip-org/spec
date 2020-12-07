# Format Implementation Guide

If you're a launcher developer, this document can help you by providing useful tips and recommended practices for implementing the MODIP Format.

---

## Implementing Frameworks

Frameworks also exist as MODIP Format Projects. It's important for launchers to understand the Project metadata and install frameworks as needed.

Framework IDs MUST be prefixed with `framework-`, so launchers can tell if a project is a Framework.

---

## Implementing Dependencies

When a user downloads a project using a launcher, it's important to download its required dependencies. Launchers should combine the version dependencies with the file dependencies in order to create a full dependency list.

For the purposes of this section, the term "the host" refers to the service a launcher is using in order to download projects, such as Diluv or Modrinth.

If a dependency's ID is `minecraft`, then it's the required Minecraft version. Implementation of where to obtain Minecraft metadata is up to the developer of a launcher. Launchers may download metadata from the host, from Mojang directly, or from another service.

If a dependency's ID is not `minecraft`, then it's a required project.

**TODO: Special case IDs may be removed.**

If a dependency has the `src` field, it means that at that specified URL is full metadata about this dependency. Launchers should check here first if it's specified. If the URL does not serve a suitable result, or there is no `src` field, launchers should then ask the host of the parent for a project matching the same ID. If at this point the launcher still hasn't found any metadata for the project, the launcher may choose to ask another host that it knows of for a project with the same ID, or it may choose to fail and warn the user.

---

## Implementing Conditions

Conditions allow values of fields to be changed based on certain variables. Conditions can only be present on certain fields.

Below is an example for modpack dependencies:

```json
"dependencies": [
  {
    "id": "client-only-mod",
    "allowed": {
      "environment": {
        "client": true,
        "server": false
      }
    },
    "required": true
  }
]
```

In the above example, if your program considers itself to be a client, then the mod should be allowed. If your program considers itself to be a server, then the mod should not be allowed to be installed (or at least without warning the user).

If the criteria for `allowed` is met (your program considers itself a client), then the mod is required to be installed. If the requirement is not meant, the mod is not required and shouldn't be installed.

Another condition you may see often is `required`
```json
"dependencies": [
  {
    "id": "client-mod-with-optional-server",
    "required": {
      "environment": {
        "client": true,
        "server": false
      }
    }
  }
]
```

If your program considers itself a client, the mod is required. If your program considers itself a server, the mod is not required to be installed.