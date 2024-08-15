# Group

## Concept

A group session is a moderated process that allows members to exchange encrypted messages securely. Each group session has a group owner, optional moderators, and group members. The workflow for managing a group session is as follows:

1. **Group Creation**
   - @Alice creates a group session named `A_family` via group factory process.
   - The session owner is set to @Alice.

2. **Session Key Management**
   - @Alice generates a session key and uploads the encrypted session key for all members to the session process (key rotation).
   - Key rotation occurs each time the members list changes.
   - Moderators, if any, can also perform key rotations.

3. **Assigning Roles**
   - @Alice can assign and withdraw the moderator role to and from members.

4. **Adding Members**
   - @Alice (or moderators) can add members directly into the group.

5. **Inviting Members**
   - @Alice (or moderators) can create a sharable link for the group.
   - Anyone with the link can request to join the group.

6. **Approving Requests**
   - @Alice (or moderators) can approve or reject join requests.
   - Upon approval, @Alice (or moderators) should also rotate the session key.

7. **Messaging**
   - Members send encrypted messages in the same manner as direct messages (DMs).

8. **Removing Members**
   - @Alice (or moderators) can remove members from the group.

## Group Factory

Responsible for spawn group session, there is only one action:

### SpawnGroup(name)

```lua
send({
    Target = "{Factory Process ID}",
    Data = "{name}",
    Tags = {
        Action = "SpawnGroup",
    },
})
```

The spawned group should store its owner

## Group Session

### InitGroup(initialSettings)

initialSettings is in json format

```ts
//group info
{
    group_name: <string>,
    about: <string>,
    img: <url>,
}

//member operation, the process should execute remove, add and set_role in order
{
    remove: [pid],
    add: [pid],
    set_role: [
        [pid, role]
    ]
}

//session key. All batches of keys should be kept in an array. The lens of array can be seen as generation number
{
    keys: [
        [pid, SK_E_pid], //pid & encrypted session key per pid
    ]
}

//initialSettings
{
    inf0: <groupInfo>,
    member_ops: <memberOps>,
    session_keys: <sessionKeys>,
}
```

```lua
send({
    Target = "{Session Process ID}",
    Data = "{initialSettings}",
    Tags = {
        Action = "InitGroup",
    },
});
```

The pids in sessionKeys should match all members' pids

### EditGroupInfo(info)

```lua
send({
    Target = "{Session Process ID}",
    Data = "{groupInfo}",
    Tags = {
        Action = "EditGroupInfo",
    },
});
```

if the name is changed, a log will be inserted into session conversion timeline

### EditMember(memberOps)

```lua
send({
    Target = "{Session Process ID}",
    Data = "{memberOps}",
    Tags = {
        Action = "MemberOps",
    },
});
```

### RequestToJoin()

Non-members can request to join a group (via a shareable url of group page)

```lua
send({
    Target = "{Session Process ID}",
    Tags = {
        Action = "RequestToJoin",
    },
});
```

the change in members should be logged into timeline

### LeaveGroup()

A member(except owner) can leave group anytime. The members list will change on leaving but the key rotation should wait until the owner or a mod is online to execute.

```lua
send({
    Target = "{Session Process ID}",
    Data = "{why you leave}"
    Tags = {
        Action = "LeaveGroup",
    },
});
```

### RotateSessionKey(sessionKeys)

```lua
send({
    Target = "{Session Process ID}",
    Data = "{sessionKeys}"
    Tags = {
        Action = "RotateSessionKey",
    },
})
```

### GetGroupMetadata()

get all the metadata of a group, including, name, img, about, members and roles. If the request entity is the owner of a mod, the return data will also include pending requests

```lua
dryrun({
  Target = "{Session Process ID}",
  Tags = {
    Action = "GetGroupMetadata",
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

Query encrypted messages and group logs, with from as start time, until as end time, limit as numbers you want to get and order as asc desc (in time). Notice that the group log is plain text but injected in timeline 

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