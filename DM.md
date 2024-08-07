# DM Session

## Concept
A DM session is a process establish between to two processes and it can only be called by those two processes. A DM session is responsible for session key management, keeping chat history. This is a workflow of encrypted DM between handleA (owned by userA) and handleB (owned by userB):

1. userA generates a session key (SK) in its local environment, outside of AO
2. userA encrypts the SK by its own pubkey and gets SK_EA in its local environment
3. userA encrypts the SK by userB's pubkey and gets SK_EB in its local environment
4. handleA sends SK_EA and SK_EB to session process
5. userA encrypts a messgae using SK, gets MSG_E
6. handleA send MSG_E to session process
7. session process sends a notification to handleB
8. when handleB is online, it checks the SK_EB and MSG_E, using userB's private key, it can decrypt SK then the MSG

## Activities

### RotateSessionKey({SK_EA, pubkey_A},{SK_EB, pubkey_B})

Update session key, can be called by either handle. All batches of keys should be kept in an array. The lens of array can be seen as generation number

```lua
send({
  Target = "{Session Process ID}",
  Data = "{pubkey_A, SK_EA},{pubkey_B, SK_EB}",
  Tags = {
    Action = "RotateSessionKey"
  }
})
```

### GetCurrentKeys()

get current session key with key generation

```lua
dryrun({
  Target = "{Session Process ID}",
  Tags = {
    Action = "GetCurrentKey",
  },
});
```

### GetKeyByGeneration(generation)

get key by generation number

```lua
dryrun({
  Target = "{Session Process ID}",
  Data = "generation",
  Tags = {
    Action = "GetKeyByGeneration",
  },
});
```

### SendMessage(content)

Send encrypted message to session process. The session will keep the message's from, pubkey, generation, timestamp with the content.

```lua
send({
  Target = "{Session Process ID}",
  Data = "{content}",
  Tags = {
    Action = "SendMessage",
  },
});
```

### QueryMessage(from, until, limit, order)

Query encrypted messages, with from as start time, until as end time, limit as numbers you want to get and order as asc desc (in time)

```lua
dryrun({
  Target = "{Session Process ID}",
  Data = "{
    from: "{from}",
    until: "{until}",
    limit: "{limit}",
    order: "{order}"
  }",
  Tags = {
    Action = "QueryMessage"
  })
```