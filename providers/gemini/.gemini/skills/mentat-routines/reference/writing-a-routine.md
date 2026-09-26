# Writing a routine's instructions

A routine's instructions are read by a harness that has just booted the agent, literally, with
nobody in the window. The three the product ships are the models; read them with `get_routine`
before writing one.

- **Second person, to the agent.** "Boot the agent this routine names …", "Read …", "Close the run".
- **Every step names something that exists** — a tool, a skill, a screen. A step that points at a
  thing the product does not have fails on every firing and tells nobody.
- **Boot and open a run first; close the run with a summary last.** The summary is the run report
  the person reads in the morning; the run is what makes every write the agent's.
- **Nobody is in the window.** What needs a person goes to the inbox through `mentat-inbox` as an
  escalation with a proposed answer, and the run is paused there. The routine never waits in the
  chat.
- **Say what it decides: nothing.** It writes, it proposes, a person chooses.
- **One thing per firing.** The Heartbeat works one row of the plan; a second row is the next
  heartbeat's.

Shape: a title; one line on when it runs and why; the standing note that the text is the person's
to change; numbered steps; the closing rule.
