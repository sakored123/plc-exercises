# PLC Exercises

Ladder Logic (LD) exercises I built while learning PLC programming with CODESYS V3.5 SP22. All projects run in CODESYS simulation mode, so no physical PLC is required.

| # | Project | What it covers |
|---|---------|----------------|
| 1 | `01_button_lamp` | Contacts, coils, series (AND) and parallel (OR) logic |
| 2 | `02_seal_in_start_stop` | Seal-in circuit, normally closed stop contact |
| 3 | `03_set_reset` | Set/reset coils, how network order changes the result |
| 4 | `04_timers` | TON and TOF timers, PT and ET |
| 5 | `05_flasher` | Two TON timers driving each other in a loop |
| 6 | `06_counter` | CTU counter: a "box full" lamp after 5 products, reset input |
| 7 | `07_counting_conveyor` | Seal-in conveyor motor that stops itself after 5 products |

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

**A counter counts edges, not levels.** Holding the sensor TRUE adds only 1, because CTU counts the FALSE → TRUE change. Q stays TRUE while CV ≥ PV, until reset.

## Counting conveyor

The counter's Q output works like a second stop button. The motor contact in series with the sensor means products are only counted while the belt is running.

```
 Network 1
     start_btn      stop_btn      sayac.Q          motor
 ──┬──| |──┬────────|/|───────────|/|──────────────( )──
   │       │
   └──| |──┘
     motor

 Network 2
     sensor       motor            sayac (CTU)        dolu_lamba
 ──────| |─────────| |──────────┤ CU         Q ├───────────( )──
                   reset_btn ───┤ RESET     CV ├──
                           5 ───┤ PV           │
```

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
