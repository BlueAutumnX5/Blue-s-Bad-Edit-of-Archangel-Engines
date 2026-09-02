# Blue's Bad Edit of Archangel Engines

Engine Simulator engines from Archangel Motors, badly edited by me. Three of them
so far, and none of them is a sensible thing to build.

Engine Simulator by [AngeTheGreat](https://github.com/Engine-Simulator/engine-sim-community-edition).
Templates by Archangel Motors from [Engine Simulator Official Discord Server](https://discord.gg/gW4SEz2R7V).

---

## Bilgewater Iron-Thumper

**997 cc, one cylinder, 4.0:1, governed to 500 rpm. Peak Torque and Power is still being discussed**

A hit and miss engine, built by the Bilgewater Iron & Steel Co. of Duckburg,
Calisota. The governor does not throttle it. It simply stops it firing until it
has slowed down enough, so the engine coasts about thirteen revolutions on its
flywheels between one thump and the next.

The strange part is that Engine Simulator can genuinely do this. Its rev limiter
re-arms on every tick the engine is over the limit and only returns the spark
once speed has fallen back below, which is a governor with hysteresis rather than
a rev limiter. The defining feature of the engine falls out of the model for
free.

There is no throttle. The Q/W/E/R keys do nothing, on purpose.

## Kaestro Boyotte

**2444 cc, one cylinder, 21.0:1, turbo diesel. About 301 Nm and 101 hp.**

One cylinder cut out of a Detroit Diesel 8V149T and converted from two stroke to
four, which is where "Quad-Cycle Single" comes from. It drives a Polski Fiat 126p
through a ZF 7DT seven speed.

It runs 21.0:1 rather than Detroit's published 17.0:1, and that is not a mistake.
A two stroke uniflow quotes its compression from liner port closure, not from
BDC, so the same untouched piston and chamber arrive at a higher number the
moment you make it a four stroke. The ratio changed because the cycle changed.

## NF300 LD

**292 cc, three cylinders, 9.0:1, redline 8000 and no rev limiter at all.**

Three cylinders of my own Indonesian Domestic Market 2003 Honda Supra Fit on a single shared crankpin, banks at
45 and 0 and minus 45. NF100 LD times three cylinders is NF300 LD.

Nothing cuts the ignition, so held wide open it runs clean past the redline to
about 12,800 rpm, the way an old bike does.

---

## Layout

| folder | what it is |
|---|---|
| `Edited Engines LTSP7/` | **The current engines.** Built on Archangel's LTSP#7.0 crate. Each has a lore file beside it. |
| `Edited Engines/` | The older LTSP#4 versions, kept so the two can be compared. |
| `Archangels-Engine-Crate-7.0/` | Archangel Motors' LTSP#7.0 crate, unchanged. |
| `Archangel Motors Crate Engine/` | Archangel Motors' LTSP#4 crate, unchanged. |
| `Story.md` | The first time I ever pushed anything to GitHub. It took four hours. |

## Getting them

Download from [Releases](../../releases), or copy the `.mr` out of
`Edited Engines LTSP7/` into your Engine Simulator `assets/engines/` folder.

Every engine has a lore file recording what it is, what was changed, and where
each number came from. Lore only dropped on the first release of new engine. the updates of these engines only drops list of changes.
