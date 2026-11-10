---
name: gavana-cli
description: Drive Gavana from a shell with the `gavana` CLI. Use when the MCP tools are unavailable, when work involves files on this machine (uploading a local image, applying a batch from a file), or when several canvas operations are better composed as commands than as separate tool calls.
---

# Gavana CLI

The CLI reaches the same Canvas Control API as the Gavana MCP tools, from the
machine you are running on. Prefer the MCP tools when they are connected. Reach
for the CLI when one of these is true:

- The work touches a **local file** — `gavana asset upload ./product.png`, or
  `gavana canvas apply canvas:<id> --file batch.json`. The remote MCP server
  runs on Gavana's side and cannot read this machine's disk.
- You have a **shell but no MCP transport**.
- Several operations compose better as commands than as separate tool calls.

## Do not memorise the command surface — ask for it

The command table is generated, so anything written down here would drift away
from it. Two commands tell you everything:

```sh
gavana capabilities          # toolsets, and which operations are paid
gavana <group> --help        # exact syntax for one group
```

`gavana capabilities` returns JSON including a `remote` array where each entry
carries `readOnly` and `paid`. Read it before acting.

## Output contract

- **stdout** is JSON and nothing else: `{"ok":true,"result":{…}}`. Parse it.
- **stderr** carries errors and progress. Never parse stdout and stderr together.
- Errors keep one shape, with `error.code`, `error.message`, and often
  `error.fields` naming what to fix.

Exit codes worth branching on:

| Code | Meaning                                 | What to do                                                  |
| ---: | --------------------------------------- | ----------------------------------------------------------- |
|  `0` | Success                                 | Continue                                                    |
|  `2` | Usage or validation error               | Fix the arguments; do not retry unchanged                   |
|  `3` | Not authenticated or not permitted      | Stop and ask the user to run `gavana auth login`            |
|  `4` | Revision conflict                       | Re-read the canvas and reapply against its current revision |
|  `5` | Not found                               | Re-resolve the handle                                       |
|  `7` | Network error                           | Retry with backoff                                          |
|  `8` | Wait timed out                          | The Run continues; resume it rather than restarting         |
|  `9` | Run reached a failed or cancelled state | Report it; do not silently retry                            |

## Spending money

Some operations are billed — image and video generation, and running a Recipe.
`gavana capabilities` marks them `"paid": true`.

Never start a paid operation the user did not ask for. If you are exploring or
inspecting, ask the user to sign in read-only instead, which removes the
possibility at the token level rather than relying on your own restraint:

```sh
gavana auth login --read-only
```

## Setup, if the CLI is missing

Gavana's connect screen tells people to paste the install line to you rather
than open a terminal, so expect to be asked to run it yourself:

```sh
npm install -g @gavana.ai/cli && gavana auth login
```

Run it when asked, and carry the whole setup rather than handing steps back.
The person pasting this was told they would not need a terminal, so every
prerequisite is yours to resolve or to explain — never a bare error.

**If `npm` is not found**, Node.js is missing. Do not install a runtime
silently, and do not just report the error either. Say what is needed in one
sentence, offer to install it, and act on the answer:

- macOS with Homebrew, or Linux with a package manager: offer `brew install node`
  or the distribution equivalent, and run it once they agree.
- Anything else, including Windows without a package manager: give them the
  download at <https://nodejs.org> and say to run the installer, then continue
  once they say it is done. This is a GUI step; do not try to script around it.

**The browser sign-in is theirs.** `gavana auth login` opens a browser and
waits. Say so and let them finish it. Do not open the URL it prints yourself —
the command already opened it, and a second `open` leaves them staring at two
tabs of the same authorization page. Pass the URL along only if they say no
window appeared.

Then confirm with `gavana auth status`. The connection shows up at `/connect` in
Gavana and is revoked from there.

Assume the person does not know what a CLI, npm, or Node.js is, and does not
need to. Report in plain terms — "Gavana is set up, I can work with files on
this computer now" — rather than pasting install logs at them.

## Before changing a canvas

Read it first and pass the revision you read back. `gavana canvas get` returns
the current revision; a stale one is what exit code `4` reports. Preserve
existing nodes and layout unless the user asked otherwise.
