# automate-termux

[LlamaLab Automate](https://llamalab.com/automate/) flows that run shell commands in
[Termux](https://termux.dev/) and get their output back.

| Flow | What it does |
| --- | --- |
| `flows/Termux.flo` | The base flow. It runs a command in Termux, waits for it to finish and returns its output and exit code. |
| `flows/Termux · uname.flo` | An example caller. It sets a command, a run mode and whether to wait for the output, then calls the base flow. Duplicate it to make your own. |

Each flow has a **README** entry point that opens this documentation in a dialog on the phone.

## Naming

Automate lists flows alphabetically. The base flow is called **Termux** and every caller
**Termux · &lt;name&gt;**:

```
Termux            ← base flow
Termux · backup   ← callers
Termux · uname
Termux · update
```

The shared prefix keeps them together, and a name that is the start of another always sorts
first, so the base flow heads the group whatever Automate's sorting rules for punctuation.
Renaming a flow in Automate does not break the callers: **Flow start** links to the base flow
by id, not by name.

## Install

1. Download the `.flo` files from the [latest release](../../releases/latest) or from `flows/`.
2. In Automate, import them with **Import** in the flow list menu. The file name becomes the
   flow name. Release downloads lose the ` · ` separator (`Termux.uname.flo`); rename those
   flows in Automate to follow the [naming convention](#naming). The name does not matter to
   the callers.
3. Do the [one-time setup](#one-time-setup).
4. In each caller, open the **Flow start** block and pick the **Termux API** beginning of
   *Termux*. Automate identifies flows by an id that is specific to each device,
   so this link cannot be shipped in the file. Pick it again after re-importing a caller.

## One-time setup

In Termux:

```bash
echo "allow-external-apps=true" >> ~/.termux/termux.properties
termux-reload-settings
termux-setup-storage
```

On Android:

- **Settings > Apps > Automate > Permissions**: allow **Run commands in Termux environment**.
  It may be listed under additional permissions.
- **Terminal mode**: give Termux the **Display over other apps** permission so it can open
  its window when it is started from the background.
- **README pages and dialogs**: Automate needs **Display over other apps** to show them
  directly instead of through a notification.
- Automate asks for file access the first time it reads a result.

## Base flow: Termux

It has three entry points:

- **Run command**: asks for a command, a run mode and whether to keep the device awake, then
  shows the output in a dialog.
- **Termux API**: for other flows. Start it with a **Flow start** block and this payload:

  ```
  {"command": "uname -a", "mode": "background", "wakeLock": 0, "reply": replyAction}
  ```

  | Key | Value |
  | --- | --- |
  | `command` | Run with `bash -c`, so pipes, `;`, `&&` and quotes work. |
  | `mode` | `"background"` (default, no window) or `"terminal"` (opens a Termux session that shows the output live). |
  | `wakeLock` | `1` keeps the device awake until the command finishes, `0` (default) does not. See [Wake lock](#wake-lock). |
  | `reply` | Optional broadcast action. When set, the result is sent back as an app broadcast with that action and the extras `{"output": text, "exit": number}`. Without it, the command runs and the result is discarded. |

- **README**: the documentation page.

`exit` is `-1` when the command did not complete: no result after 600 s, or Termux could not
be started.

### How it works

1. Automate starts Termux's `RunCommandService` with `bash -c`. The script runs the command,
   writes stdout and stderr to `/sdcard/Download/automate-termux-<time>.txt.tmp`, appends
   the exit code, then renames the file without `.tmp`. Automate therefore never reads a
   half-written file. In terminal mode, `tee` also shows the output in the session.
2. Automate checks for the file every second, for up to 600 s.
3. It reads the file, deletes it, and splits it into the output and the exit code.
4. It sends the result back to the caller, shows it (**Run command**), or discards it.

The reply is an app broadcast restricted to Automate's own package, so other apps do not
receive the output.

### Wake lock

With `wakeLock` set to `1`, the flow keeps the CPU and Wi-Fi awake from the start of the
command until its result is ready, then lets the device sleep again. It uses Automate's
**Device keep awake** block, whose lock belongs to that run only, so several runs do not
release each other's lock.

Termux's own wake lock (`termux-wake-lock`, or *Acquire wakelock* in its notification) is
never acquired or released by these flows. If it is on, it stays on, whatever `wakeLock` is.
A run with `wakeLock` set to `0` changes nothing: the device may sleep only if nothing else
keeps it awake.

## Callers: Termux · …

The settings are the first four **Variable set** blocks:

| Variable | Value |
| --- | --- |
| `command` | The bash command. In an Automate string, `{…}` inserts an expression; write `\{` for a literal brace. |
| `mode` | `"background"` or `"terminal"`. |
| `wakeLock` | `1` to keep the device awake until the command finishes, `0` not to. |
| `waitForOutput` | `1` to get the result back in this flow, `0` to only start the command (nothing is shown). |

With `waitForOutput = 1`, the caller makes up a unique broadcast action, passes it as
`reply`, and waits on **Broadcast receive**. The result ends up in two variables:

- `output`: what the command printed (stdout and stderr).
- `exitCode`: the exit code, or `-1`.

The last block only shows them in a dialog. Replace it with whatever should use the result.

## Limits

- Interactive commands (`top`, `read`…) never finish in background mode. Use terminal mode.
- A command that runs longer than 600 s returns `exit = -1`. Its partial output stays in
  the `.tmp` file.
- The caller starts listening for the reply right after it starts the base flow. The base
  flow takes at least a second to answer, so the reply is not missed in practice.

## Editing the flows

The `.flo` files open in [Automate Web Builder](https://xosh.org/AutomateWebBuilder/), a
desktop-sized editor for Automate flows. Use **Open** on the `.flo` file. Do not go through
its JSON import: that turns variables and expressions into plain text and breaks the flow.

## Acknowledgements

The flows were generated with the library of
[Automate Web Builder](https://github.com/SMUsamaShah/AutomateWebBuilder) (MIT), which reads
and writes Automate's `.flo` format.

## License

[MIT](LICENSE)
