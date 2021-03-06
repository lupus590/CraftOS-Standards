# *COS num:* File System over Rednet

> Currently just a copy paste of tror as I'm using it as a template - @Lupus590

## Quick information
| Information |                                                                |
| ----------- | -------------------------------------------------------------- |
| Version     | 1.0.0                                                          |
| Type        | Protocol                                                       |

## Technical Details
File System over Rednet, or FSoR, defines a method of bidirectional file
transmition. Additionally it defines ways for a client to request additional
information about the server's file system, such as a list of the files.

> May need to change stuff here - @Lupus590
The server and client communicate via a series of "packets". Packets are
generally sent with one packet per rednet message, but multiple packets MAY
be sent at once with each being delimited with the line feed (LF) character
(`\n`, ASCII code 10). Consequently, the packet's payload CANNOT contain the LF
character.

Packets are composed of 3 components:

 - A two character packet code. This defines the command to execute. An unknown
   command MUST be discarded.
 - Optional implementation-specific data about the packet. Arbitrary data MAY be
   placed here but SHALL NOT contain a semicolon (`;`) or a LF character. The
   implementation MUST NOT malfunction if data is found here.
 - The packet's payload.

These components are combined in the format:
```
<Code>:<Metadata>;<Payload>
```

The packet MUST take this exact format even if one component is empty. Both
`:` and `;` MUST be present.

 > Might be a good idea to change these so tror can be used with fsor - @Lupus590
### Connection management and security
The server MAY send packets to the client to handle the connection, such as
determining what extensions to the protocol the client supports.

| Code | Payload         | Description                                         |
| ---- | --------------- | --------------------------------------------------- |
| `SP` | `<extensions>`  | Carries the extensions supported by the server and requests a capability list from the client. Each extension should be surrounded in hyphens. This packet MAY be sent at any time. |
| `SC` | `<message>`     | Closes this connection. The server SHOULD NOT send any more packets after this one, nor handle any incoming packets. |
| `PW` | `<encrypt/decrypt techneques supported by the server>` | Requests the client to send a username and password. This packet MAY be sent at any time. |
| `DE` | `<reason>`      | Sent when the server wishes to deny permissions for a request, such as trying to write to a read only file. Or for bad username password combinations. Meta data can be the code of the message which instigated this responce. |
| `AK` | None | Sent to acknowledge something. Meta data can be the code of the message which instigated this responce.

Some packets require the client to respond.

| Code | Payload         | Description                                         |
| ---- | --------------- | --------------------------------------------------- |
| `SP` | `<extensions>`  | Carries the extensions supported by the client. Each extension should be surrounded in hyphens. This packet MAY be sent at any time and SHOULD be sent whenever a `SP` packet is received. |
| `PW` | `<username>,<password>` | Meta data can include the encryption techneque used. The username and password may both or neither be encrypted or any combination of the two. If no encryption is used then meta should indicate but servers should be able to handle no meta. This packet may ONLY be sent in responce to reciving a `PW` packet.

### File browsing packets
The following packets are sent to the server from the client so that the client
can explore the remote file system.

| Code | Payload         | Description                                                                 |
| ---- | --------------- | ----------------------------------------------------------------------------|
| `LS` | `<directory path>` | Request a list of files in the directory path.                           |

The server should send responces to the packets as follows.

| Code | Payload         | Description                                                                 |
| ---- | --------------- | ----------------------------------------------------------------------------|
| `LS` | `<directory path>` | Request a list of files in the directory path.                           |







| `TY` | `<f,b,t>`       | Sets the foreground, background and text of the current line. All three fields MUST be the same length.\*†                       |


† All fields being the same length allows the packet parser to read `n`
characters once the first `,` has been found. The implementation SHOULD discard
the packet if each field is not the same length but MAY attempt to recover.

### Client querying commands
The server MAY send packets to the client to determine information about the
client such as color capabilities. The client SHOULD send an appropriate
response packet.

| Code | Payload         | Description                                                   |
| ---- | --------------- | ------------------------------------------------------------- |
| `TQ` | None            | Queries the client for terminal dimensions and color support. |

The client should send an appropriate response packet to these requests:

| Code | Payload         | Description                                         |
| ---- | --------------- | --------------------------------------------------- |
| `TI` | `<w>,<h>,<col>` | Carries the client terminal's width, height and color support to the server. This packet MAY be sent at any time and SHOULD be sent whenever a `TQ` packet is received. |

### Client events
The client MAY send events such as key presses to the client. However the server
MAY ignore these.

| Code | Payload             | Description                                     |
| ---- | ------------------- | ----------------------------------------------- |
| `EV` | `<event,arg1,arg2>` | Queues an event on the server.                  |

The event name and its arguments should be displayed as the appropriate nil,
boolean, string or number literal. Strings must use double quotes and escape the
LF character.

Lua tables MAY be included as an argument, though an implementation is allowed to
discard it. Other Lua expressions such as binary operators or functions MUST NOT
be transmitted and SHOULD NOT be executed.

### Extension: Text table
The text table extension allows sending a serialized version of the terminal
buffer using the TRoR protocol. This buffer contains information about
cursor status, terminal size and terminal contents.

Support of this extension SHOULD be shown by including the `textTable` in the
`SP` packet. This extension MUST NOT be used if the client does not indicate its
support and `TV` should be used instead.

| Code | Payload         | Description                                         |
| ---- | --------------- | --------------------------------------------------- |
| `TT` | `<payload>`     | Sets the entire terminal's state.                   |

The payload should be a serialized table and MUST NOT contain the LF character.
The table MUST contain the following fields:

| Field         | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| `sizeX`       | The width of the terminal. This MUST be an integer.          |
| `sizeY`       | The height of the terminal. This MUST be an integer.         |
| `cursorX`     | The cursor's X position. This MUST be an integer.            |
| `cursorY`     | The cursor's Y position. This MUST be an integer.            |
| `cursorBlink` | Whether the cursor is blinking. This MUST be either `true` or `false`. |
| `text`        | A table representing the textual contents of the terminal.†  |
| `textColor`   | A table representing the background color of the terminal.\*†|
| `backColor`   | A table representing the textual contents of the terminal.\*†|

† Each entry of the table represents a separate line of the terminal. There must
be `sizeY` table entries, each being `sizeX` characters long.

## Available Utilities
 - [nsh with the put and get addon utilities](https://github.com/lyqyd/cc-netshell/)
 - [lnfsd](https://github.com/lyqyd/lnfs-daemon) and it's companion [lnmount](https://github.com/lyqyd/lnfs-client)
 - [Unnamed FTP Server with client API](https://github.com/CC-Hive/FTP)

