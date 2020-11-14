# Format Implementation Guide

If you're a launcher developer, this document can help you by providing useful tips and recommended practices for implementing the MODIP Format.

---

## Implementing Frameworks

Frameworks also exist as MODIP Format Projects. It's important for launchers to understand the Project metadata and install frameworks as needed.

Framework IDs SHOULD be prefixed with `framework-`. This naming scheme makes it clear that it is no "ordinary" mod, though not being treated any different.

---

## Implementing Dependencies

When a user downloads a project using a launcher, it's important to download its required dependencies. Launchers should combine the version dependencies with the file dependencies in order to create a full dependency list.

For the purposes of this section, the term "the host" refers to the service a launcher is using in order to download projects, such as Diluv or Modrinth.

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

---

## Multi-file zips
Users may want to import a MODIP multifile zip. This zip contains a root file, `index.modip.json` which contains metadata about the project, which is likely to be a modpack/instance. Parse the project the same way you would as a normal project, except that when encountering a file name, check to see if it exists in the zip. If you encounter a file with the `name` field set to `overrides.zip`, check for a file named `overrides.zip` inside the multifile zip. If you find one and it's hash matches succesfully, you don't need to download it from the files `downloads` array.
## Implementing `game`
When parsing a version of a project, the `game` field specifies which Minecraft version it's compatible with.

When parsing the `versions` field, the lowest version is considered the minimum. Any version released before the lowest version was released, the project is considered to not function at all.

For versions above the highest version listed, it's up to you to decide what to do. You might consider maintaing your own "compat database" of which versions are compatible. But that's up for you to decide.

### FAQ
**Q: Where do I get Minecraft version metadata, if it's not specified anywhere?**  
A: Get it from Mojang at http://launchermeta.mojang.com/mc/game/version_manifest_v2.json

**Q: How do I compare Minecraft versions?**
A: Compare using release dates. Use the `releaseTime` field in the Mojang version manifest specifically. It may sound counterintuitive, when it's easy to compare versions like 1.16.1 and 1.16.2 together, but it lets you compare snapshots and old versions easily.
