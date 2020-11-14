# Format Implementation Guide

If you're a launcher developer, this document can help you by providing useful tips and recommended practices for implementing the MODIP Format.

---

## Implementing Frameworks

Frameworks also exist as MODIP Format Projects. It's important for launchers to understand the Project metadata and install frameworks as needed.

Framework IDs MUST be prefixed with `framework-`, so launchers can tell if a project is a Framework.

---

## Implementing Dependencies

When a user downloads a project using a launcher, it's important to download its required dependencies. Launchers should combine the version dependencies with the file dependencies in order to create a full dependency list.

When looking at a dependency, the first thing to do is check for the `src` field. If the `src` field exists, then the mod has external MODIP metadata. Process the metadata for that dependency from the `src` field.

If there is no `src` field, you have two options:
1) Request information on this ID from hosts that your launcher knows of.
2) Throw a warning to the user, suggesting that they need to download the project, but the launcher cannot automatically install it.
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