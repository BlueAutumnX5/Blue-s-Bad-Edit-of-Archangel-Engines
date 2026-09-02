# Sabar Type H

A **hit and miss** engine. One cylinder, 997 cc, governed to 500 rpm, driving a
workshop line shaft off its own flywheel by flat belt.

*Sabar* is Indonesian for **patient**. It fires about once every thirteen
revolutions and spends the rest of its life coasting, which seemed worth naming
it for. *Type H* for hit and miss.

## What a hit and miss engine is

Before throttles were any good, stationary engines held their speed with a
flyball governor that did something wonderfully crude: once the engine was
turning fast enough, the governor **stopped it firing**. It then coasted on its
flywheels - five revolutions, ten, twenty - until it had slowed enough for the
governor to drop back in, fired once or twice, sped up, and was cut off again.

Hence the name, and hence the sound:

```
bark ...... ...... ...... bark bark ...... ...... ...... bark
```

An engine that is idle most of the time and flat out the rest of it. There is no
throttle anywhere on it. The mixture is fixed and the governor is the only speed
control it has.

## Why Engine Simulator can genuinely do this

Every other engine in this repo is a fight against something the simulator will
not model. This one is the opposite, and it is luck rather than cleverness.

engine-sim's rev limiter is **not** a fixed-length spark cut. From
`src/ignition_module.cpp`:

```cpp
if (m_enabled && m_revLimitTimer == 0) { /* ...spark... */ }

m_revLimitTimer -= dt;
if (std::fabs(m_crankshaft->m_body.v_theta) > m_revLimit) {
    m_revLimitTimer = m_limiterDuration;
}
```

The timer is re-armed to its full duration on **every tick the engine is above
the limit**, and the spark is only allowed back `limiter_duration` seconds
*after* speed has fallen below it again. That is a governor with hysteresis -
cut until slow enough, then a dwell - which is precisely the mechanism of a
spark-cut hit and miss governor.

So the defining behaviour of this engine is not a stand-in for anything. It falls
out of the model for free. `E_omegamax` is the governed speed and `LICO_t` is the
length of the miss.

## The engine

| | |
|---|---|
| Layout | single cylinder, four-stroke, air-cooled |
| Bore × stroke | 100.0 × 127.0 mm (stroke is exactly 5.00 in) |
| Displacement | 997.5 cc / 60.9 cu in |
| Compression | 4.0:1 |
| Governed speed | 500 rpm |
| Output | ~21 Nm, ~1.4 hp, 2.6 bar BMEP |
| Valves | 2 × 38 mm - one atmospheric intake, one cam-driven exhaust |
| Ignition | make-and-break igniter, fixed 20° BTDC, no advance |
| Flywheels | 500 mm, 46.7 kg for the pair, 89% of all rotating inertia |
| Fuel | gasoline, rich (lambda ~0.88) through a fixed mixer |

Mean piston speed at the governed 500 rpm is **2.12 m/s**, about a fifth of a
modern engine. That is why a thing like this can be built out of cast iron and
babbitt and still be running eighty years later.

### The atmospheric intake valve

The intake valve has no cam, no pushrod and no lobe on the camshaft. It is held
shut by a light spring and pulled open by the piston's own suction. engine-sim
cannot express a valve that opens itself, so it is modelled by putting the events
where the physics would put them:

- It **opens after TDC** (`IV_O` is negative). It cannot open before, because
  before TDC there is nothing to open it - the cylinder is still full of exhaust
  at or above atmospheric.
- It **shuts at the bottom of the stroke**, because the rising piston pushes it
  onto its seat. It cannot be held open into compression the way a cammed valve
  is.
- The **overlap is exactly zero**. This is forced, not chosen: a self-opening
  valve cannot be asked to open against a cylinder that still has exhaust
  pressure in it.

It also lifts less than the exhaust valve (6 mm against 8), because vacuum can
only drag it so far against its spring.

## Measured, not estimated

Cold start, this file as committed, no load:

| | |
|---|---|
| Cadence | fires at 481-495, then coasts ~13 revolutions, repeat |
| Speed range | roughly 464-579 rpm around the governed 500 |
| Coast-down rate | 60-62 rpm/s |
| Total drag during a miss | 10.7 Nm (4.6 friction, 6.1 pumping) |
| Gear 4, brake held | 393-549 rpm, still hit-and-missing, never stalls |

Peak torque was **not** taken from the dynamometer - that UI is close to unusable
on an engine this slow, for reasons written up in `ENGINE-NOTES.md`. It came from
the engine's own dynamics instead, which is better evidence anyway: the rotating
inertia is known exactly (1.643 kg·m²), so the energy the flywheel gives up
during a miss and puts back during the hits gives 2.6 bar BMEP and ~21 Nm
directly.

1.4 hp from a litre is exactly the historical rating for an engine of this
class, and 2.6 bar BMEP sits right in the 2-4 bar band a low-compression 1900s
stationary engine belongs in.

It overshoots its own governed speed by up to 15%. That is not a fault: the
limiter tests the **instantaneous** crank speed, and a single cylinder with a
127 mm stroke speeds up hard on the power stroke and slows on compression, so it
trips on a within-cycle peak rather than on the mean. On a big slow single the
governed speed is a band, not a line.

### The one thing that had to be tuned

`LICO_t`, the length of the miss. The limiter's dwell is a fixed **time**, not a
speed, so the engine always fires later the harder it happens to be
decelerating. A real flyball governor re-engages on speed and does it instantly.
This is the one place engine-sim genuinely mismodels the mechanism, and it is
invisible until something is hanging off the flywheel.

| `LICO_t` | unloaded, fires at | braking in 4th, fires at |
|---|---|---|
| 2.0 | fine | **fired once and died, every gear** |
| 1.0 | 438-456 | 357-390 |
| **0.35** | **481-495** | **393-427** |

At 2.0 s every overshoot bought a mandatory two second blackout, and under load
the engine was dragged to a standstill inside it and could never earn its spark
back. At 0.35 it fires just under the governed speed, which is what the real
thing does, and the miss is still long because most of it is the engine decaying
off its own 577 rpm overshoot rather than the timer running.

**It should not go much lower.** One four-stroke cycle at 500 rpm takes 0.24 s,
and the limiter tests the *instantaneous* crank speed, which dips below the limit
during every compression stroke. Once the dwell is shorter than a cycle the timer
expires inside those dips, the spark returns, and the governor leaks: the engine
just keeps firing above its governed speed. 0.35 sits deliberately just above one
cycle.

## The drivetrain, which the template gets wrong three times

Everything below the flywheel had to be rebuilt, because the template's formulas
assume a car:

**The clutch** was the worst of them. `CL_Nmax = E_Nmax * 1.33` assumes peak
torque is about a third above the mean, which is true of a multi-cylinder engine
whose power strokes overlap. This engine fires once every thirteen revolutions:
its mean is 21 Nm but the measured peak is **158 Nm**, a pulse ratio of 7.5:1. So
the clutch was rated at a sixth of what actually arrives and slipped on every
single hit. Nothing downstream can feel right while that is happening. It is now
a flat 250 Nm.

**The brake** came out of a formula that sizes it to stop a car from 100 km/h in
40 m, giving 1447 N against an engine that can only put 160 N through the belt in
the top step. It was up to nine times stronger than the entire engine, so
touching it stalled the motor outright. Now 350 N, which is a shop friction brake
on the countershaft: a wooden block on a strap.

**The gearing** spanned 12.0:1 down to 2.37:1 in 1.5 steps, with over 1000 N
available in bottom. Wide steps are pointless here because the engine has no rev
range to drop into. It is now a proper five step cone pulley in true 4:3 steps,
6.0:1 down to 1.9:1.

## The line shaft

There is no vehicle. engine-sim requires one, so the "vehicle" is the shop: the
flat belt runs on the face of the flywheel itself (which is why the wheel
diameter is set to 500 mm and the tyre is zeroed out), `FD_R` is the main belt
down to the line shaft, and the five "gears" are the five steps of a cone pulley
on the countershaft - which is genuinely how a shop of this period changed the
speed of a machine. You stop, you lever the belt across a step, you start again.

## The story

*Not written yet - this file records what the engine is and where its numbers
come from. The narrative belongs to BlueAutumnX5.*
