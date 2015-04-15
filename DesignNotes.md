# Change Monitoring #

Google Docs provides a changes feed for the user, which can be used to get a list of all changes since a given _changestamp_. A changestamp is a unique, monotonically increasing integer representing each change to a users docs list. Each entry in a feed has a changestamp value that uniquely and globally identifies a specific change to the relevant resource. We must store the value of the latest changestamp that we have consumed, and then query the API from that changestamp in the future. This process is repeated over time to detect changes.

A changestamp is only valid until the associated resource is changed again. When that happens, all previous changestamps for the resource become invalid. This results in gaps between changestamps when the changes feed is pulled. To detect if a specific resource has changed, GET the self link of the change. A 404 response to this request indicates that the resource has since changed; a new changestamp is now associated with the resource and the original changestamp is no longer valid. Otherwise, the change entry is returned.

Entries in the changes feed are ordered in ascending chronological order. That is, the oldest changes show up first. To collect all changes, follow all next links in the returned feed until there is no next link returned.

To get the changes feed for the currently authorised user, use the URL:
https://docs.google.com/feeds/default/private/changes

Google Docs has 2 modes of indicating element deletion in change entries:
  * `gd:deleted` means that the resource was put in the trash, just as it does in a normal resource entry.
  * `docs:removed` means that the change was a permanent deletion of the resource from the documents list.

# Renaming and Deletion #

I don't have any clear idea yet how to handle renames and deletes.

# Caching & Synchronisation #

What we really want is to store files locally, and keep them synchronised with the gdrive backend. We'll want to be able to to configure our driver to specify which parts of the gdrive backend we're going to sync with; some people have way too much stuff in there...

It's ok if the cache is not always in sync with the backing store; we simply want best effort.

A few thoughts:

  * Writing; we write to local storage, and queue an update to the backing store. To be robust, we'd want to log the update request to a journal so that it could be pushed to the backing store if the filesystem was unmounted/aborted and then remounted.
  * Opening; first check with the backing store to ensure we have the latest version. If not, pull down the latest. This may knock performance, and may not be essential.
  * Ideally, we'd want the backing store to provide us with some means of retrieving a version/checksum/etc. of a directory, to give us an idea if that dir or any subdir were the same version as what we were caching. When the filesystem is mounted, we could traverse the tree of all dirs we sync with to ensure we were in sync, and pull down any changes. Likewise, we could poll for changes once the filesystem was up.
  * The backing store must provide us with a version or checksum for each file.

Questions:
  * Where do we store file metadata? Could we use extended attributes for this?

# Conflict Resolution #

Implement some pluggable interface, so we can mess with different methods.

Rules;
  * We must never pull down changes to an open file.

Possible strategies;
  * Assume one client only ever accesses a file; we always write back to the file in the backing store. For this to work, we need to check that we always have the latest version of the file when we open it.

# Other Questions #

  * Files can appear in multiple collections (folders). Do we try to handle this with symlinks in order to minimise disk usage, or just maintain copies? It's not clear if folders can appear in multiple collections.

  * There are two cases where a resource may not be linked to a visible parent collection and will not appear in the "root" collection. In both of these cases, the resource still appears in the user's feed. The first case is when a resource is shared with a user but not placed in a collection. In this case, the resource appears in the user's feed, and does not appear in any collection, including the "root" collection. The second case is when a resource is shared with a Google Apps domain, not placed in a collection, and opened up for viewing by the user. In this case the resource is in no collections, including the root, but appears in the user's feed.