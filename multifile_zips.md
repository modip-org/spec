# Multifile zips

It's useful to be able to store a collection of multiple projects inside one zip file. The most obvious example of this is for storing modpacks. To properly store multiple in one zip, the proper architecture as listed below should be used:

```
my_modpack.zip/
  index.modip.json
```

The metadata for the project should be stored inside `index.modip.json`. The file's name MUST be `index.modip.json`, otherwise clients will have no idea where to look for proejct metadata.

What if you want to store other files, like an overrides zip for a modpack? That can be done by simply storing the file, with it's name matching the `name` field in `version.files`.

```
my_modpack.zip/
  index.modip.json
  overrides.zip
```

While reading the project's metadata, clients will notice the file `overrides.zip`, and then check for it inside the parent zip file. If it's found, then it will use the local `overrides.zip` instead of trying to download one from a URL specified in `file.downloads`. Because of this, you MAY leave the `downloads` array blank, but it is still recommended to provide download URLs if possible.

## Why?
This is most useful for modpacks. If you'd like to send a modpack to a friend, for example, multifile zips are the best way to do it. They keep the entire pack in one file, and allow bundling of other files within them, making implementation easier.