## Creating a NFS Share

These are the options currently in use:

`all_squash, anongid=100, anonuid=1001, insecure, no_subtree_check, rw, sync`

**- all_squash**
Maps all client users (including root) to the anonymous user (specified by anonuid and anongid). Effectively makes every client request run as that one user.

**- anonuid=1001**
Sets the UID of the anonymous user to 1001 (e.g., qbtuser). This is the UID that all client requests get mapped to with all_squash.

**- anongid=100**
Sets the GID of the anonymous user to 100 (e.g., users). Similar to anonuid, this sets the group for all squashed client requests.

**- insecure**
Allows clients to connect from non-privileged ports (ports > 1024). Some clients use high ports, so this relaxes the default security restriction that requires privileged ports.

**- no_subtree_check**
Disables subtree checking. This improves reliability and performance by not verifying the exact path of a file within the exported directory for each access.

**- rw**
Grants read-write permissions to clients for this share.

**- sync**
Requires the NFS server to write changes to disk before replying to the client, ensuring data integrity at the cost of some performance.

>NOTE: anonuid, anongid and all_squash are mapped to qbtuser on the seedbox in order to get write permission.
