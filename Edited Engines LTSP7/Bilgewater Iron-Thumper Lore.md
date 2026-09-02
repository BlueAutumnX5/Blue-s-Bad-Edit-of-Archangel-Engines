# Bilgewater Iron-Thumper

A **hit and miss** engine. One cylinder, 997 cc, governed to 500 rpm, driving a
workshop line shaft off its own flywheel by flat belt.

Built by the **Bilgewater Iron & Steel Co.** of **Duckburg, Calisota**, who made
castings for anybody who asked and engines for anybody who insisted. The
Iron-Thumper was their smallest, and the only one of their products that ever
left the state in numbers.

The dashboard says **BISCo Iron-Thumper**, because the foundry stamped its
castings BISCo the way Fairbanks-Morse stamped F-M and International Harvester
stamped IHC. It is also all that fits: the simulator appends the displacement to
the name, and past about 20 characters the two run into each other. Everywhere
outside the sim it is the Bilgewater Iron-Thumper in full.

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
| Output | 78.5 Nm and 5.5 hp at the governed 500 rpm, 9.9 bar BMEP |
| Valves | 2 × 38 mm - one atmospheric intake, one cam-driven exhaust |
| Ignition | make-and-break igniter, fixed 20° BTDC, no advance |
| Flywheels | 500 mm, 46.7 kg for the pair, 91% of all rotating inertia |
| Fuel | gasoline, rich (lambda ~0.88) through a fixed mixer |

Mean piston speed at the governed 500 rpm is **2.12 m/s**, about a fifth of a
modern engine. That is why a thing like this can be built out of cast iron and
babbitt and still be running eighty years later.

### The flywheels, and why the template gets them wrong

Checked against real engines of this exact size. A 1938 **John Deere 1.5 HP**
runs 17.5 inch flywheels and weighs 226 lb (103 kg) all up; another 1.5 HP of the
period uses 25 inch wheels with a 2.25 inch rim; a **Galloway 1.5 HP** ships at
140 lb. On a roughly 100 kg engine the flywheels are typically getting on for
half the total weight.

So 500 mm and 46.7 kg for the pair is right where it should be, and sits inside
the real 445 to 635 mm range.

**The inertia was not right.** The template computes a solid disc:

    FW_MOI = (1/2) * m * r^2        ->  1.459 kg.m2

A flywheel is not a disc. It is a spoked wheel with almost all of its iron in the
rim, deliberately, because putting the mass at the largest possible radius is the
entire reason the thing exists. Splitting it the way a real casting is built,
85% in the rim at 0.88 of the outer radius and 15% in hub and spokes at 0.35,
gives **1.975 kg.m2** instead. The template was understating the real flywheel by
**35%**.

That is not a detail on this engine. The flywheel is 91% of all rotating inertia,
and the inertia is exactly what decides how long it coasts through a miss. The
correction slows the coast from 62 to 47 rpm/s, which makes the whole thing
lazier and more like the real article.

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
| Torque, on the dyno | 53.8 Nm at 200 rpm, 62.2 at 300, 70.7 at 400, 78.5 at 500 |

Torque climbs all the way to the governed speed, so the governor happens to cut
in exactly at peak. Every point is consistent on `P = T * omega`.

### The peak torque figure was wrong for a while, and how

This file first claimed **21 Nm and 1.4 hp**. It came from the engine's own
dynamics rather than the dyno, because the dyno looks unusable at this speed:
take the energy the flywheel gives up during a miss, divide by the number of
firing strokes in the burst that follows, and you have the work per cycle.

The method is sound. The firing-stroke count was not. It came from tachometer
samples half a second apart, which cannot resolve a burst lasting a fraction of a
second, and guessing 3.8 strokes where there were nearer 1.2 divides the answer
by three.

What made it convincing was a coincidence. 21 Nm and 2.6 bar is *exactly* what a
real Fairbanks-Morse Z 1.5 HP makes, so a wrong number landed on an independent
expectation and that read as confirmation. The lesson is that agreeing with what
you expected is not evidence.

Properly latched, the dyno gives **78.5 Nm and 5.5 hp**.

### It is about 3.7 times as strong as the real thing

9.9 bar BMEP is roughly 23 percent brake thermal efficiency. A real engine of
this size and era made 2.6 bar and 1.5 hp.

That gap is the simulator being generous, not the file being wrong. engine-sim
has **no knock model**, so running 4.0:1 compression costs it almost nothing,
while on a real 1910 engine the low compression is most of the reason it only
made a horsepower and a half. The compression is correct, the fuel is correct,
and the simulator simply does not charge for it.

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
whose power strokes overlap. On a single that fires once every thirteen
revolutions it is not: the measured peak is **158 Nm** against a mean of 78.5, a
pulse ratio of about **2:1**, and a good deal more than that instant to instant
because the 158 is itself a filtered reading.

At the time the clutch was fixed `E_Nmax` was still the wrong 21 Nm, so the
formula was rating it at 28 Nm against a 158 Nm peak and it slipped on every
single hit. Nothing downstream can feel right while that is happening. With the
corrected 79 Nm the formula would give 105, which is still under the peak. Either
way the answer is the same: **size the clutch off peak, not off the mean.** It is
now a flat 250 Nm.

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
diameter is 500 mm and the tyre is zeroed out), the main belt to the line shaft
is 1:1, and the five "gears" are the five steps of a cone pulley on the
countershaft. That is genuinely how a shop of this period changed the speed of a
machine: you stop, you lever the belt across a step, you start again.

| step | ratio | belt speed |
|---|---|---|
| 1 | 1.848 | 25 km/h |
| 2 | 1.307 | 36 km/h |
| 3 | 0.924 | 51 km/h |
| 4 | 0.653 | 72 km/h |
| 5 | 0.462 | **100 km/h** |

Root-two steps, a spread of 4.0. The fast step is where the belt finally runs at
the speed it was built for: 100 km/h is 27.8 m/s, and real flat belts run at 15
to 30 m/s.

### The vehicle "weight" is 11 kg, and that is correct

It looks absurd next to a 100 kg engine. The trap is reading `V_M` as the weight
of a machine. It is not. `transmission.cpp` turns it into a **rotating inertia**,
`I = V_M * f * f`, so what it actually describes is the inertia of whatever the
engine spins, and **a 100 kg engine bolted to a concrete block contributes
exactly zero**. Only the turning parts count.

For a real small-shop line shaft those are: a 50 mm shaft ten metres long, which
is only 0.05 kg.m2 because its radius is tiny, and four 500 mm pulleys at 25 kg
each, which are 3.1. About **3.2 kg.m2** all told, and in the fast step that
needs `V_M` of about 11 kg.

It had been 400, then 150, then 80, dropped each time because the engine could
not pull the shop. Every one of those was too high for the same wrong reason:
they were being read as weights. 80 kg was producing 23.4 kg.m2, seven times a
real line shaft, which is why the engine took minutes to wind it up.

### How the speed readout actually works

Worth writing down, because it is not what it looks like. engine-sim reports
speed by energy equivalence:

```cpp
const double E_r = 0.5 * m_rotatingMass->I * v_theta * v_theta;
return std::sqrt(2 * E_r / m_mass);
```

`Vehicle::getSpeed` never mentions the tyre radius, which suggests the radius is
irrelevant and the vehicle mass matters. Both are wrong. `transmission.cpp` sets
that virtual body up as

```cpp
const double f = tire_radius / (diff_ratio * gear_ratio);
m_rotatingMass->I = m_car * f * f;
m_rotatingMass->m = m_car;
```

so the mass cancels straight back out and the whole thing collapses to
`speed = omega_engine * f`. It is plain kinematics after all: **the vehicle mass
has no effect on speed whatsoever**, and the tyre radius sets it completely.
`V_M` only decides how much inertia and rolling drag the engine has to fight.

Checked before it was trusted: the previous 1.899 overall read 23 km/h at 467
rpm, against 23.2 predicted.

## The story

Bilgewater Iron & Steel never set out to build engines. They built castings, and
they built them for whoever walked into the yard at Duckburg with a drawing and
cash. Somewhere around the turn of the century a customer failed to collect an
order of flywheels, and rather than weigh them in, the foundry built engines
around them. That is the whole origin of the Iron-Thumper and it explains its one
distinguishing feature, which is that the flywheels are far too big for it.

Half the weight of the engine is in those two wheels. It is also the reason the
thing works at all.

Every shop that ever had one has the same story. Somebody new walks in, hears it
miss, and says the engine is finished. It has been turning for the best part of a
minute without firing: the flywheels going round, the exhaust just breathing, the
whole thing apparently coasting to a death. Then it thumps twice, picks up, and
goes quiet again. The new person stands there waiting for it to stop.

It does not stop. It is not idling either, because it has no idle. It is waiting,
with 47 kilos of Calisota iron already turning, for a reason to fire, and the
governor decides when that is. Nobody else gets a vote. There is no throttle to
get wrong, no timing to creep, no electronics to drown. There is a weight in the
flywheel, an igniter with two contacts, and a mixer with a needle you set once a
season and then leave alone.

It burns roughly what it needs and nothing more, because the governor simply
refuses to let it fire when it does not have to. That was the whole idea and it
was a good one: an engine that idles by not running.

One and a bit horsepower, which sounds like nothing until you notice it has been
one and a bit horsepower every working day since before anyone in the building
was born. Through the line shaft it drives whatever is belted up that week.

The flywheels deserve room. They carry it through thirteen silent revolutions to
the fourteenth loud one, and they are turning at waist height with nothing around
them, which is why every one of these engines has a story attached to it and why
the good ones are bolted to something that will not move. Bilgewater sold them
without guards and considered the matter closed.

Fifteen hundred revolutions an hour, most of them silent, and one thump in
fourteen. It will outlast whoever bought it. Most of them have.

---

*Written by Claude at BlueAutumnX5's request, to his setting, and entirely his to
rewrite. The numbers above it are measured. This part is not.*
