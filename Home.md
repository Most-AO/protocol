# MostAO

Messages and other stuff transmitted by AO

## Background

AO, the computer, is a promising new infrastructure that enables Web3 applications to deliver a Web2 experience. To enhance the AO ecosystem, a protocol is required to establish a standardized messaging system for processes to communicate.

## Conceptes

- Handle: A handle is the unique identifier, and is also a process that can send and receive messages.
- Session: A session is process establish between two handles. A session is responsible for keeping the message history and key exchange. 
- Registry: Register all handles and spawns sessions between handles.

## Handle Registry

Every process in MostAO has a (social) handle as its unique identifier. The handle records are managed by a handle registry process. The activities of the registry process is as follows:

### QueryHandle(name)

Returns the process id of this handle.

```ts
dryrun({
  Target = "{Registry Process ID}",
  Data = "{name}",
  Tags = {
    Action = "QueryHandle",
  },
});
```

### GetHandles(owner)

Return the handles under this owner (wallet address)

```ts
dryrun({
  Target = "{Registry Process ID}",
  Data = "{owner}",
  Tags = {
    Action = "GetHandles",
  },
});
```

### Register(name)

Register a handle name, spawn a handle process with this name in it

```ts
send({
  Target = "{Handle Process ID}",
  Data = "{name}",
  Tags = {
    Action = "Register",
  },
});
```

### QuerySession(handleA, handleB)

Return session id if there exists a session between handleA and handleB (order does not matter), otherwise return nil.

```ts
dryrun({
  Target = "{Registry Process ID}",
  Data = "{handleA, handleB}",
  Tags = {
    Action = "QuerySession",
  },
});
```

### EstablishSession(otherHandle)

This function can only be called by either one of handle. It will spawn a session process between the handleA and the handleB. Then notify the two handles to update the chatlist.

```lua
send({
  Target = "{Registry Process ID}",
  Data = "{otherHandle}",
  Tags = {
    Action = "EstablishSession"
  }
})
```

## Handle

A handle process acts as an agent for a user. Handle has those functions:

1. keep a profile along with env variables
2. keep chat list
3. relay messages of its owner
4. send and receive messages

The activities of the handle process is as follows:

### ProfileUpdate(profile)

Update the profile of this handle, the structure of profile data is as follows:

```ts
{
  name: <string>,
  img: <url>, //profile picture,
  banner: <url>,
  bio: <string>,
  pubkey: <string>,
  ...other fields
}
```

The pubkey field is the wallet pubkey. But in the future, MostAO will support multiple pubkey scheme, allowing different communication model.

```ts
send({
  Target = "{Handle Process ID}",
  Data = "{profile}",
  Tags = {
    Action = "ProfileUpdate",
  },
});
```

### GetProfile()

Retrieve the profile of this handle.

```ts
send({
  Target = "{Handle Process ID}",
  Tags = {
    Action = "GetProfile",
  },
});
```

### RelayMessage(WrappedMessage)

Send wrapped message from user (user's wallet) to handle. Then the handle will relay this message to target process with its inner data. Only the owner of handle can call it.

The WrappedMessage is in json format
```
{
  Target: "{Target Process Id}",
  Data: "{Data}",
  Tags: {
    Action: "{Action}",
  },
}
```

```lua
send({
  Target = "{Session Process ID}",
  Data = "{WrappedMessage}",
  Tags = {
    Action = "RelayMessage",
  },
});
```

### UpdateChatList(sessionInfo)

This function is called by the Registry process. It updates the chat list of this handle with new session information. Only the Registry process is authorized to call this function.

```lua
send({
  Target = "{Handle Process ID}",
  Data = "{sessionInfo}",
  Tags = {
    Action = "UpdateChatList"
  }
})
```

### GetChatList()

This function is called by the handle owner. It retrieves the chat list of this handle, including all related session information.

```ts
send({
  Target = "{Handle Process ID}",
  Tags = {
    Action = "GetChatList",
  },
});
```

### Notify(data?)

Whenever a handle sends a message to session process, the session process will notify the conterpart of incoming message by sending a notification to it. This can only be called by session process. The data is optional for now.

```lua
send({
  Target = "{Handle Process ID}",
  Data = "{data}",
  Tags = {
    Action = "Notify"
  }
})
```

