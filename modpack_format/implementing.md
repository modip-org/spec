# Implementing the Modpack Format
This is a guide for launchers on how to implement the modpack format.

## Format Meta
Format Meta, like `formatType` and `formatVersion` are meta-fields used for describing what format the file is in.

### `formatType`
If you encounter an unknown `formatType`, an error should be thrown and shown to the user. Installation of a modpack cannot continue of the `formatType` is unknown.

### `formatVersion`
You may want to add a special indicator if future versions are encountered in `formatVersion` - for example, the `formatVersion` field of a modpack is set to `2`, while the launcher only supports version `1`. As this is an unknown format to the launcher, you may want to make it clear with a message that it's an unsupported future version.

## Files
Loop through each file in the `files` array. You may want to do this in batches (e.g. 5 files at a time, concurrently) for efficiency.

### `path`
Each path of the file is relative to the Minecraft instance directory. For example `mods/MyMod.jar` resolves to `.minecraft/mods/MyMod.jar`. When parsing the `path` field, make sure it doesn't exit the Minecraft instance directory for security reasons.

### `sha256`
This hash should be checked after downloading, to verify that the file downloaded is the same file as what was supposed to be downloaded. If the hashes do not match, a warning should be shown to the user about this mismatch.

### `env`
To parse `env`, you must first know whether or not you are a client or a server. For launchers, they are most likely to be a client. Server-side software is most likely a server. 

If this field exists, check if the field for your side (`client` or `server`) exists and is set to `true`. If it's not set to `true`, then you are not supposed to download this file. You may ignore it, and you don't have to show a message to the user.

### `optional`
Collect a list of optional files while parsing the `files` array. Show a dialog to the user that allows them to select which optional files they would like to install.

### `downloads`
In order to download files, you must loop through this array. Go from top to bottom. Attempt to download a URL, and if it doesn't file and the downloaded file matches the SHA256 hash, the file has been sucessfully downloaded. If the file was unable to download or the hash didn't match, move on to the next URL in the list.