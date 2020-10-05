# Format Implementation Guide

If you're a launcher developer, this document can help you by providing useful tips and recommended practices for implementing the MODIP Format.

---

## Rendering HTML

The MODIP Format Specification allows the usage of HTML in certain fields, such as `description` and `changelog`. HTML is widely used and meets many of the requirements needed for a field like `description`. However, HTML can be **very dangerous**. Take caution when displaying content using HTML. It's recommended to sanitize your HTML according to the rules listed below.

### Disallowed Tags

**TODO: Blacklists are a bad idea. Needs to be changed to whitelist.**
You should never attempt to render any of the following tags: `script`, `style`, `link`, `title`, `button`, `form`, `input`, `meta`, `option`, `textarea`, and `select`.

### iframes

`iframes` can be used, but it's recommended to only render them when the hostname is from a trustworthy source. Trustworthy sources are up to implementation by the launcher.

### Scripting

Forms of client side code execution, such as JavaScript and WebAssembly, _should never be allowed_. XSS attacks are dangerous - do not allow untrusted code to run in your launcher.

### Sanitization

It's recommend to whitelist tags and attributes based on conventions from a host. For example, if the host you're integrating your launcher with only uses `<p>` tags, only allow `<p>` tags to be rendered. It's a better approach that minimizes any potential risk.

Make sure to sanitize all attributes. For example, XSS attacks can use the `onerror` attribute to run JavaScript code after loading an invalid image. Only allow the attributes that are required.

### Frequently Asked Questions

**Q: I've received HTML in a field that doesn't allow for it in the specification. Should I parse it anyway?**  
A: No. Do not attempt to parse HTML outside of a field that it's allowed in. If you receive a string, `<p>My Mod</p>` in the `name` field, render it exactly as written, including the HTML tags. Don't attempt to strip HTML from a value.

**Q: I've received Markdown. Should I parse it?**  
A: Markdown is not allowed in the specification. You shouldn't attempt to parse it. Hosts should be providing HTML, not Markdown.

**Q: I don't want to render HTML at all. Should I just strip it out?**  
A: If for whatever reason you do not want to render HTML, then yes, the best method is to remove the tags. Keep in mind the consequences of not rendering HTML, as users may lose helpful and useful features such as images and video embeds.

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