# Bilgewater Iron-Thumper

A **hit and miss** engine. One cylinder, 997 cc, governed to 500 rpm, driving a
workshop line shaft off its own flywheel by flat belt.

Built by the **Bilgewater Iron & Steel Co.** of **Duckburg, Calisota**, who never
intended to build engines at all.

The dashboard says **BISCo Iron-Thumper**, because the foundry stamped its
castings BISCo the way Fairbanks-Morse stamped F-M and International Harvester
stamped IHC. It is also all that fits: the simulator appends the displacement to
the name, and past about 20 characters the two run into each other. Everywhere
outside the sim it is the Bilgewater Iron-Thumper in full.

## What a hit and miss engine is

Before throttles were any good, stationary engines held their speed with a
flyball governor that did something wonderfully crude: once the engine was
turning fast enough, the governor **stopped it firing**. It then coasted on its
flywheels, five revolutions, ten, twenty, until it had slowed enough for the
governor to drop back in, fired once or twice, sped up, and was cut off again.

Hence the name, and hence the noise:

```
thump ...... ...... ...... thump thump ...... ...... ...... thump
```

An engine that is idle most of the time and flat out the rest of it. There is no
throttle anywhere on it. The mixture is fixed and the governor is the only speed
control it has.

## Why Engine Simulator can genuinely do this

Every other engine in this repo is a fight against something the simulator will
not model. This one is the opposite, and it is luck rather than cleverness.

engine-sim's rev limiter is **not** a fixed length spark cut. From
`src/ignition_module.cpp`:

```cpp
if (m_enabled && m_revLimitTimer == 0) { /* ...spark... */ }

m_revLimitTimer -= dt;
if (std::fabs(m_crankshaft->m_body.v_theta) > m_revLimit) {
    m_revLimitTimer = m_limiterDuration;
}
```

The timer is re-armed on **every tick the engine is above the limit**, and the
spark only comes back `limiter_duration` seconds *after* speed has fallen below
it again. That is a governor with hysteresis, which is precisely the mechanism of
a spark-cut hit and miss governor.

So the defining behaviour of this engine is not standing in for anything. It
falls out of the model for free. `E_omegamax` is the governed speed and `LICO_t`
is the length of the miss.

## The engine

| | |
|---|---|
| Layout | single cylinder, four stroke, air cooled |
| Bore x stroke | 100.0 x 127.0 mm (the stroke is exactly 5.00 in) |
| Displacement | 997.5 cc / 60.9 cu in |
| Compression | 4.0:1 |
| Governed speed | 500 rpm |
| Output | 78.5 Nm and 5.5 hp at the governed speed, 9.9 bar BMEP |
| Valves | 2 x 38 mm, one atmospheric intake and one cam driven exhaust |
| Ignition | make and break igniter, fixed 20 deg BTDC, no advance |
| Flywheels | 500 mm, 46.7 kg for the pair, 91 percent of all rotating inertia |
| Mean piston speed | 2.12 m/s, about a fifth of a modern engine |

### The atmospheric intake valve

The intake valve has no cam, no pushrod and no lobe on the camshaft. It is held
shut by a light spring and pulled open by the piston's own suction. engine-sim
cannot express a valve that opens itself, so the events are placed where the
physics would put them:

- It **opens after top dead centre**, because before that there is nothing to
  open it. The cylinder is still full of exhaust at or above atmospheric.
- It **shuts at the bottom of the stroke**, because the rising piston pushes it
  back onto its seat.
- The **overlap is exactly zero**. Forced, not chosen: a self opening valve
  cannot be asked to open against a cylinder that still has exhaust in it.

It also lifts less than the exhaust valve, 6 mm against 8, because vacuum can
only drag it so far against its spring.

## Measured, not estimated

Cold starts, full application restarts, never through RELOAD.

| | |
|---|---|
| Cadence | fires at 475 to 495 rpm, then coasts about 13 revolutions |
| Speed range | 464 to 579 rpm around the governed 500 |
| Coast down rate | 41 rpm/s |
| Total drag during a miss | 10.7 Nm (4.6 friction, 6.1 pumping) |
| Gear 4, brake held | 393 to 549 rpm, still thumping, never stalls |
| Top of the cone pulley | 99 to 101 km/h of belt speed |
| On the dynamometer | 53.8 Nm at 200 rpm, 62.2 at 300, 70.7 at 400, 78.5 at 500 |

Torque climbs all the way to the governed speed, so the governor happens to cut
in exactly at peak.

It is about **3.7 times as strong as the real article**, which managed 1.5 hp
from the same litre. That gap is the simulator being generous rather than the
file being wrong: engine-sim has no knock model, so running 4.0:1 costs it almost
nothing, while on a real 1910 engine the low compression is most of the reason it
made so little. Working notes for everything else, including how the drivetrain
was rebuilt and why the flywheel inertia had to be corrected, are kept separately
and not in this file.

---

## The story

Bilgewater Iron & Steel never set out to build engines.

They were a waterfront foundry, down where the Duckburg quays run out into the
mudflats, and for thirty years they made exactly what a harbour needs. Bilge
pumps, which is where the name comes from and which nobody in the family ever
found funny. Capstans. Bollards. Deck fittings by the ton. Anchor stocks for
ships that mostly never came back. If you could draw it and pay for it, the yard
on Kettle Street would pour it.

Then in the winter of 1903 a mill upriver ordered forty flywheels, half a metre
across, and went under before a single one was collected.

The castings sat in the yard for two years. They were too good to break up and
too specific to sell, and old Bilgewater would not weigh in anything with his
mark on it. So when his son came back from the coast talking about the gas
engines that were starting to appear in the canneries, the answer to *what would
we build it around* was already stacked against the north wall, going orange in
the rain.

That is the whole origin of the Iron-Thumper, and it explains its one obvious
feature. The flywheels are far too big for it. Half the weight of the engine is
in those two wheels, on a motor that makes about as much power as a large dog.

It turned out to be the best mistake the yard ever made.

All that iron meant the engine could coast a dozen revolutions and barely notice,
and once you can coast, the governor does not have to be clever. It does not need
to meter anything or throttle anything. It only has to decide whether to let the
next spark happen. A weight, a spring, and a trip lever. Nothing else on the
engine moves at all between one thump and the next.

Everything after that was decided by a foundry rather than by engineers, which is
to say it was decided by what was cheapest to cast and hardest to break. The
intake valve has no cam because a cam costs money and the piston will pull the
valve open for free. Both valves are the same 38 mm casting because one pattern
is cheaper than two. Compression is 4.0:1 because that is all the igniter and the
fuel of the day would stand, and because the yard's own foreman is on record
saying he would rather build it twice as heavy than half as strong.

They sold a few hundred, mostly along the Calisota coast, to sawmills and
creameries and cannery sheds and anywhere a line shaft needed turning and nobody
wanted to think about it again for twenty years.

Every one of those places has the same story. Somebody new walks in, hears it
miss, and says the engine is finished. It has been turning for the best part of a
minute without firing: the flywheels going round, the exhaust just breathing, the
whole thing apparently coasting to a death. Then it thumps twice, picks up, and
goes quiet again. The new person stands there waiting for it to stop.

It does not stop. It is not idling either, because it has no idle. It is waiting,
with 47 kilos of Calisota iron already turning, for a reason to fire.

The flywheels deserve room, and Bilgewater sold them without guards and
considered the matter closed. They are turning at waist height with nothing
around them, which is why every one of these engines has a story attached to it,
and why the good ones are bolted to something that will not move.

Fifteen hundred revolutions an hour, most of them silent, and one thump in
fourteen. The yard on Kettle Street closed in 1948. The engines did not.
