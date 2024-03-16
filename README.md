# VCRemoteCommand

This library provides a wrapper around SwiftNIO SSH for interacting with remote computers through SSH and SFTP. It features an SFTP library and full PTY terminal support.

# Usage

As SSH is a multichannel protocol, before one can begin interacting with clients, they need to create a connection to use for the client.

```
let connection = RCConnection(host: ip, port: port, username: username, password: password)
try await connection.connect()
```

Once, a connection is established to the remote server, you can create either a shell, sftp, or ssh client. 

## Shell client

A shell client is useful for creating a PTY terminal. 

```
let client = try await connection.createShell()
client.onReceived = { bytes in
    print("Got back \(bytes)")
}
if let lsCmd = "ls\n".data(using: .utf8) {
    try await self.shell?.send(ArraySlice(cdCmd))
}
```


## SFTP client

A sftp client is useful for modifying and listing files on a remote server. 

```
// Writing to a remote file
let data = "Test writing".data(using: .utf8)
let client = try await connection.createSFTPClient()
try await client.write(data, file: path)
```

```
// Listing a remote directory
var response = try await client.list(path: path)

// Directories can have multiple pages of files
let shouldClose = response.hasMore
while response.hasMore {
    response = try await client.append(to: response)
}
if shouldClose {
    try await client.close(response: response)
}
print("\(path) has: \(response.files)")
```


