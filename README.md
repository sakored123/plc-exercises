# PLC Exercises

Ladder Logic (LD) exercises I built while learning PLC programming with CODESYS V3.5 SP22. All projects run in CODESYS simulation mode, so no physical PLC is required.

| # | Project | What it covers |
|---|---------|----------------|
| 1 | `01_button_lamp` | Contacts, coils, series (AND) and parallel (OR) logic |
| 2 | `02_seal_in_start_stop` | Seal-in circuit, normally closed stop contact |
| 3 | `03_set_reset` | Set/reset coils, how network order changes the result |
| 4 | `04_timers` | TON and TOF timers, PT and ET |
| 5 | `05_flasher` | Two TON timers driving each other in a loop |

## Key lessons

**Contacts read, coils write.** A value written by a coil is overwritten on every scan, so it cannot be changed from outside.

**The stop contact must be on the shared path.** In my first seal-in circuit the stop contact sat inside the parallel branch. The motor kept running through the seal-in contact and could not be stopped. Moving the stop contact after the branch joins fixed it:

```
     start        stop         motor
 ──┬──| |──┬──────|/|──────────( )──
   │       │
   └──| |──┘
     motor
```

**Network order matters.** The PLC runs networks from top to bottom, so when set and reset are both active, the last network wins.

## Flasher logic in Structured Text

```
t_off(IN := alarm AND NOT t_on.Q, PT := T#2S);
t_on(IN := t_off.Q, PT := T#500MS);
lamp := t_off.Q;
```

## Running a project

1. Open the `.project` file in CODESYS V3.5.
2. Select **Online → Simulation**.
3. Log in with `Alt+F8` and start with `F5`.
4. Set a prepared value and write it with `Ctrl+F7`.
