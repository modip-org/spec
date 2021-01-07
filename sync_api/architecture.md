MODIP is planned to be two things: a specification for a mod index, and a
central repository which actually holds metadata for all Minecraft mods,
formatted according to the specification.

The central repository is not the only repository - other people may also
publish repositories which aren't necessarily related to the central one.

The Sync API is an API designed to allow a server of one repository to import
the entirety of another repository.

# Repository linking

Repositories should support mirroring other repositories. 

Two modes:

* Mirror - simple mirror, always keep up-to-date with the other repository
* Import - import all projects in the other repository into this one, optionally under a namespace prefix.

Note that the namespace prefix only affects the human-readable IDs of projects. Projects may also be identified by some
type of SHA256 hash which does not change when importing.

Note that every repository is responsible for the content on that repository. Creating a project does not guarantee
that all other repositories will mirror it accurately. Repositories are encouraged to add additional information to project pages,
if they have such information and they cannot get it accepted upstream.

# Overall network structure

We envision that:

* Other people may want to run central repositories.
* Groups of mod developers may want to use the same format to publish their mods. Central repositories can pull from it.
* Launcher developers may want to mirror a central repository, perhaps in a more efficient format for their launcher.

Overall, this forms a directed graph, including cycles, of repositories which import from each other. New content may be
introduced into any "upstream" repository and it will propagate to all other interested repositories.

# Trust model

We assume that a repository will not be configured to import from any other repositories which its administrator does not trust.

However, malicious projects will inevitably get introduced anyway, so it's important to be able to trace them.

The namespace prefix system allows one to simply identify the origin of a project. If Repo A has a project with the name
"repo_b:repo_c:mod_name" then it clearly came via Repo B (directly) and Repo C (indirectly).
Repo A's administrator may blacklist the project, but they should also work with Repo C's administrator to blacklist
the project at the source, or Repo B to stop importing from Repo C, if they are uncooperative.

Note: "malicious" here refers to spam, mods that harm computers or worlds, mods that impersonate other authors,
and so on. Repositories are **strongly discouraged** from blacklisting mods over simple disputes such as mods which
refuse to run based on the current player's UUID. However, they are free to add warning labels, or links to patched
versions of the mods.

# Use cases

## Upstream from central repository

**If** there ends up being a central MODIP repository, the Sync API allows the
central repository to import metadata directly from the original authors of the
mods, provided the authors are willing to provide it in this format.

## Downstream from central repository

Systems which allow users to pick and choose mods to install (hereby "modpack
generators") may find it useful to have their own copy of the metadata for all
available mods, for quick access.

"Leaf" systems (those with no repositories downstream of them) can also
reformat the data to better fit their internal data structures.

## Few central repositories

If there ends up being more than one central repository, they may pull data
from each other using the API, in order to ensure they have the most complete
data set to present to users.

It is recognized that central repository operators may want to prevent other
repositories from accessing their data, in order to compete by having more
data. It is also recognized that this would be dickish behaviour, and other
participants should respond by cutting off their access in turn.

## Peer-to-peer web

If there is no central repository, we may end up with a loose mesh of peer
repositories operated by various individuals. Provided that enough connectivity
exists across the mesh, links between repositories should be sufficient to
propagate information about any project all across the web, in a
web-of-trust-style model. Malicious repositories may find their access cut off
by other participants.

# Other goals

* It should be possible to implement the API as a static file server.
  Therefore, there should be no query-string parameters (`?foo=bar&baz=quux`).

# Is this a blockchain?

no
