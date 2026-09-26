# The scheduler on this machine

## The command

`mentat schedule` lives in the plugin's command-line tool. On Claude Code it is run from the plugin
itself; elsewhere from the published mirror:

| Harness | Run it as |
| --- | --- |
| Claude Code | `node "${CLAUDE_PLUGIN_ROOT}/cli/bin/mentat.mjs" schedule …` |
| Codex, and every tree harness | `pnpm dlx github:razvanpiticas/mentat-plugin schedule …` |

Node 22.12 or later is needed, which the plugin already needs.

```
mentat schedule add --harness <claude-code|codex> --routine <id> --name "<name>" --cron "<m h dom mon dow>" --zone "<IANA zone>" --project-dir "<path>" --prompt "<text>"
mentat schedule remove --routine <id>
mentat schedule list [--project-dir "<path>"]
```

`add` refuses: a harness other than the two above ("no verified non-interactive command"); a zone
that is not the machine's; a line Task Scheduler cannot express (Windows only); a second entry for
the same routine (remove first). It prints the entry it wrote and when it next fires. `remove`
finds the entry by the routine id it stamped and prints what it removed, or "nothing installed" —
never an error for an entry that is not there. `list` prints every entry the command made, one
line each: routine id, harness, line, project directory, what runs.

## What is installed

| Machine | Entry |
| --- | --- |
| macOS, Linux | one crontab line: `<m h dom mon dow> cd "<project-dir>" && <command> # mentat:<routine id>`, appended with `crontab -l` / `crontab -` so the person's other lines stay |
| Windows | one Task Scheduler task per hour in the line, named `Mentat <routine id>` or `Mentat <routine id> <n>`: `schtasks /Create /SC DAILY /ST HH:MM /TN "<name>" /TR "cmd /c <script>" /F`, or `/SC WEEKLY /D <days>` for a weekday list. The `<script>` is one `.cmd` file per routine under `%LOCALAPPDATA%\Mentat\schedule\`, holding the directory change and the harness command; `remove` deletes it with the tasks. Task Scheduler takes at most 261 characters for what a task runs, and a project path plus the boot prompt is past that, so the task runs the script and the script carries the rest |

The command per harness:

| Harness | Command the entry runs |
| --- | --- |
| Claude Code | `claude -p "<prompt>" --permission-mode auto --permission-prompts none` |
| Codex | `codex exec "<prompt>" --sandbox workspace-write` |

Task Scheduler limits: minutes and hours must be numbers or lists; day of week `*` or a list of
days; day of month and month must be `*`. Steps (`*/2`), day-of-month lists and month values are
refused: "simplify the line or run it by hand".

A Task Scheduler task holds a frequency and one time of day, not a cron line, so `list` on Windows
prints the task's own schedule — `DAILY 08:00`, `WEEKLY MON,TUE,WED,THU,FRI 09:30` — where a
crontab prints the five fields back. The routine's line as the server holds it is in
`list_routines`, which is where to read it.

## What must already be true

- The harness has signed in to Mentat once on this machine, in this project's directory, in an
  interactive session — the plugin's sign-in flow. A scheduled run that finds no token fails
  without saying so.
- On Claude Code, the project directory is one the plugin is enabled in, and the routine was run
  once by hand so the tool approvals are saved; `--permission-prompts none` denies anything that
  would have prompted. **That flag needs Claude Code 2.1.259 or later**; an earlier one rejects the
  whole command with `error: unknown option '--permission-prompts'` and the routine fires into
  nothing. `claude --version` says which this machine has.
- The machine is on and awake at the time. A Desktop task catches up one missed run; a cron or
  Task Scheduler entry does not.

## Symptoms

| Symptom | Cause | Do |
| --- | --- | --- |
| The routine is on, the run report never comes | no entry on this machine, or the machine slept | `schedule list`; install; keep the machine awake |
| The run report says it could not sign in | no cached token for a run nobody started | run it once by hand in the harness, then let the schedule fire |
| The run stalls | a tool prompted and nobody answered | on Desktop, *Run now* and *always allow*; in a terminal, the `--permission-prompts none` flag is missing from the entry |
| Fires at the wrong hour | the routine's zone is not the machine's | set *Read in* to the machine's zone and reinstall |
| `schedule add` refuses the line on Windows | Task Scheduler cannot express it | simplify the line, or run by hand |
| `error: unknown option '--permission-prompts'` | Claude Code is older than 2.1.259 | update Claude Code; the entry needs no change |
