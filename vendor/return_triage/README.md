# Return Triage

Route inspected returns to stock, repair, or disposal while preserving the return authorization ID.

Commerce and fulfillment · framework · 0.1.1

## Drive the workflow

Start with `return_triage_start()`. For each event call `return_triage_step(snapshot, event, value)`, persist the returned snapshot, and dispatch `return_triage_command(snapshot)` to your application’s adapter. The host owns I/O, event ordering, duplicate detection and durable command delivery. A matching event produces one command. Rejected events retain the phase and saved quantity. An invalid snapshot yields an empty vector.

The snapshot contains phase ID, saved quantity and last command ID. Event values are whole numbers from zero through one billion. In the table, **x** is the event value and **r** is the saved quantity. Rules match in listed order. To validate a whole history without running effects use `return_triage_replay(events, values)`. It returns the final snapshot or an empty vector on rejection.

| Current phase | Event | Guard | Next phase | Command | New quantity |
|---|---|---|---|---|---|
| idle | authorize | x>0 | transit | send_label | x |
| transit | receive | x=r | inspect | quarantine | r |
| inspect | grade | x=0 | closed | restock | r |
| inspect | grade | x=1 | repair | request_repair | r |
| inspect | grade | x=2 | closed | recycle | r |
| repair | complete | x=r | closed | restock | r |

Phase IDs in order from zero: idle, transit, inspect, closed, repair.

## Replay your own events

After installing the demo, run the following from its directory. Supply alternating event names and integer values. The command prints each requested action and exits with status 1 at the first rejected transition, or 2 for malformed arguments.

```sh
blyxc native-run main.blyx -- authorize 9 receive 9 grade 1 complete 9
```

## Try the published project

```sh
nest demo return_triage@0.1.1
blyxc native-run return_triage-demo/main.blyx
```

Install with `nest add return_triage@0.1.1` and import `vendor::return_triage::return_triage`.

Maintained by Rahul Chaube. [Blyx](https://www.blyx-lang.space) · [GitHub](https://github.com/Rahulchaube1) · [LinkedIn](https://www.linkedin.com/in/rahulchaube1) · [Sponsor](https://github.com/sponsors/Rahulchaube1)
