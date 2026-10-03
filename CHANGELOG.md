# Changelog

## v1.1.0

- Rename the flows so that Automate's alphabetical list groups them, with the base flow
  first: **Termux run command** becomes **Termux**, and callers are named
  **Termux · &lt;name&gt;** (the example becomes **Termux · uname**). The README pages
  inside the flows use the new names.
- README: add a naming section.

Existing installs only need the flows renamed in Automate; callers keep their link to the
base flow.

## v1.0.0

First release.

- **Termux run command**: runs a shell command in Termux in the background or in a
  terminal session, and gets its output and exit code back. Entry points: **Run command**
  (interactive), **Termux API** (for other flows, with an optional broadcast reply) and
  **README**.
- **Termux uname**: an example caller, with the command, the run mode and the choice to
  wait for the output in separate Variable set blocks.
