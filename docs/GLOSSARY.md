# Attachment uploading

This plugin uploads files a note references and points those references at the returned address.

## Language

**Attachment**:
A vault file referenced by a note.
_Avoid_: asset, media file

**Remote image**:
An image a note references by an HTTP(S) URL, rather than as a vault file.
_Avoid_: attachment, external link

**Upload command**:
The executable invocation a user configures to send one local file.
_Avoid_: shell pipeline, hosting API

**Destination URL**:
The HTTP(S) address an upload command reports for one uploaded file.
_Avoid_: output, result, link

**Recovery receipt**:
A local record of a destination URL that was not written into the note.
_Avoid_: backup, remote deletion

**Batch migration**:
A user-started pass over notes in the vault.
_Avoid_: sync, background scan

**Skip prefix**:
A URL prefix the user lists so matching remote images stay unchanged.
_Avoid_: whitelist, domain filter

**Session record**:
A destination URL remembered until Obsidian quits.
_Avoid_: cache, history

## Relationships

- A note references many **Attachments** and **Remote images**
- An **Upload command** reports one **Destination URL** for one local file
- A **Destination URL** takes the place of the reference it came from
- A **Recovery receipt** records a **Destination URL** the note did not receive
- **Batch migration** visits notes that are not open, and only files a note references
- A **Skip prefix** applies to **Remote images** and survives a restart
- A **Session record** suppresses uploading that **Destination URL** again during the same Obsidian run, and is cleared when the selected upload tool or its **Upload command** changes. A **Skip prefix** is kept
