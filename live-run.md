# Live run for openchamber/openchamber#7607

Web on Linux. `bun run dev` from this branch (c0bc806f3) and from `main` (3853549df), each in an isolated instance with its own HOME and data folders, OpenCode 2.4.1, model Big Pickle.

Session "Rebase PR 42" is asked to show an `awaitAnswer` page titled "How should I resolve the README conflict in PR 42?" and wait. In session "Check on PR 42", an agent is told to call `session.status` on it, then `session.send` it "Keep ours for the README conflict.", and report the JSON. The JSON below is the tool output stored in OpenCode's database, trimmed to the fields that matter.

## A page, and an agent's send

`main`:

```json
{"action":"session.status","data":{"sessionStatus":{"type":"waiting"}}}
{"action":"session.send","data":{"promptDispatched":true,"dispatchedAsCommand":false}}
```

The agent then told the user: "The session was waiting for your input and has now received the instruction to keep our side of the README conflict."

This branch:

```json
{"action":"session.status","data":{"sessionStatus":{"type":"waiting","waitingOn":[{"kind":"page","title":"How should I resolve the README conflict in PR 42?"}]}}}
{"action":"session.send","data":{"promptDispatched":true,"dispatchedAsCommand":false,"waitingOn":[{"kind":"page","title":"How should I resolve the README conflict in PR 42?"}],"note":"The session is waiting for the user to answer what waitingOn lists, so your prompt waits in its inbox, and its agent reads it only after the user answers there. Send the user to that session to answer; don't report the prompt as delivered."}}
```

The agent then told the user: "The session is still waiting for someone to answer its README-conflict question in that session; your instruction is queued in its inbox and won't be acted on until that answer is given."

On both, the prompt sat in the waiting session's inbox (`session_inbox`, `delivery: steer`, with the sender's `promptedBy`) and the page stayed open.

## An agent's slash command to the page

Another agent sent `/review` to the same session:

```json
{"action":"session.send","data":{"promptDispatched":true,"dispatchedAsCommand":true}}
```

No `waitingOn`: the command reaches OpenCode without the sender's metadata, and the page's call returned `{"answered":false,"reason":"chat"}`.

## A question form

```json
{"sessionStatus":{"type":"waiting","waitingOn":[{"kind":"form","title":"Questions","questions":["Which name should the 2.8 release use?"]}]}}
```

Sends from the control route, with no calling session, of "Use Aurora." and then `/review`:

```json
{"promptDispatched":true,"waitingOn":[{"kind":"form","title":"Questions","questions":["Which name should the 2.8 release use?"]}]}
{"promptDispatched":true,"dispatchedAsCommand":true,"waitingOn":[{"kind":"form","title":"Questions","questions":["Which name should the 2.8 release use?"]}]}
```

Eight seconds later both prompts, the command's expanded text included, were still in the inbox as steers, and the form was still open.

## A shell permission

The session's rules ask before any shell command. Its status, then a send of "Also show the last commit." from the control route:

```json
{"sessionStatus":{"type":"waiting","waitingOn":[{"kind":"permission","action":"shell","resources":["git status"]}]}}
{"promptDispatched":true,"waitingOn":[{"kind":"permission","action":"shell","resources":["git status"]}]}
```

Eight seconds later the prompt was still in the inbox and the permission still pending.

## A fork of the waiting session

A fork of the session waiting on the form, with a prompt:

```json
{"action":"fork","sourceSessionId":"ses_debdfc48fffeWuRVCUqikVft1f","promptDispatched":true}
```

No `waitingOn`. The fork answered at once and went idle.
