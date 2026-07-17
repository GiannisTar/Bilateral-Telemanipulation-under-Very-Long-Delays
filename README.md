# Bilateral Telemanipulation under Very Long Delays

A wave-variable based control scheme for force-feedback teleoperation over communication delays of up to 2 seconds round-trip — the kind of delay you'd see controlling a robot on the Moon or Mars, or over a poor network link. Diploma (integrated Bachelor's/Master's) thesis, Department of Computer Science and Engineering, University of Ioannina, supervised by Prof. Kostas Vlachos, 2022.

## At a glance

| | |
|---|---|
| **Problem** | A human operator controls a remote ("slave") robot and feels its contact forces through a haptic device ("master") — but round-trip delay alone is enough to make a naive force-feedback controller oscillate and go unstable |
| **Approach** | Wave variables (Niemeyer & Slotine): encode force and velocity into a single passivity-preserving signal, so stability no longer depends on the size of the delay |
| **Side effects tackled** | Position drift (master and slave positions diverge over time) via "Adjusting the Wave Command"; wave reflections (residual energy causing oscillation) via impedance matching; free-motion vs. contact behavior via online tuning of the wave impedance |
| **Testbed** | Simulated mass-and-friction master/slave models, then a real 3-DOF Novint Falcon haptic device over ROS 1 |
| **Delay tested** | Up to 2 seconds round-trip (1s RTT used as the main baseline, inspired by real ISS↔ground-station space-teleoperation delays) |
| **Extension** | A velocity-control mode for the master side, trading position-tracking for a practically unbounded workspace |
| **Result** | Stable, passive teleoperation in free motion, in contact with an obstacle, and combinations of both — in simulation, with added friction, and on real hardware |

## Why wave variables

Force-feedback teleoperation over a delayed channel has an obvious failure mode: without any correction, feedback forces combined with the operator's own input drive the master and slave into growing oscillations, and the system becomes unstable — this gets worse the longer the round-trip delay is. Wave variables (introduced by Niemeyer & Slotine in 1991) sidestep this by transforming the force and velocity at each end into a single "wave" signal — a bijective, invertible transform, so no information is lost, but the transmission itself becomes provably passive regardless of the delay. As long as a system is passive, it's guaranteed stable. Only the wave signal needs to cross the communication link; each side decodes it into whatever it needs — the slave turns an incoming wave into motion in free space or into force when in contact, the master turns a returning wave into the feedback force applied to the operator's hand.

This passivity guarantee doesn't come for free: the wave transform introduces two side effects of its own, position drift and wave reflections, both of which this thesis addresses and tests.

## Dealing with position drift and wave reflections

**Impedance matching** counters wave reflections by making both the master and slave subsystems present as a pure damper to the wave signal, so incoming wave energy is dissipated instead of bouncing back and forth. It's implemented with two tuned damping terms (one for a critically damped response, one to match the controller's impedance to the wave impedance), which — as derived and simplified in the thesis — collapses the whole controller into a simple, well-understood block diagram.

**Adjusting the Wave Command** counters position drift by feeding the slave's position back to the master and adding a small corrective term to the outgoing wave, proportional to the accumulated position error — without needing a second communication channel or breaking passivity.

**Online tuning of the wave impedance** lets the same controller behave well in very different tasks: a low wave impedance favors free, agile motion (matching velocity), while a high impedance favors accurate force transmission (good for pushing against something, or feeling a hard collision) — tunable on the fly as the task changes between free space and contact.

## Experimental setup

The physical setup is a Novint Falcon, a 3-DOF haptic controller connected over USB, interfaced through ROS 1 (Noetic, on Ubuntu 20.04) via the `libnifalcon` driver. Since the control scheme is single-DOF, two of the Falcon's three axes are locked with a simple PID controller and only the forward/backward axis is teleoperated. The main delay used throughout testing is 1 second round-trip, chosen to roughly match the communication delays reported for real DLR↔ISS space-teleoperation missions (ROKVISS, KONTUR-2) — with the system also verified stable up to 2 seconds RTT.

Testing proceeds in three stages: first a full simulation (simple mass models for master and slave, in free space and in contact with an immovable object), then the same experiments repeated with viscous + Coulomb friction added to the slave model for realism, and finally the same experiments again with the real Novint Falcon replacing the simulated master.

## Extending to an unbounded workspace: velocity control

The Falcon's own workspace is small (about 20 cm), which limits the standard position-matching controller to short-range or fine manipulation tasks. As an extension, the thesis reworks the master side so that the master's *position* relative to a home point maps to the slave's commanded *velocity* instead — giving a practically unlimited operating range (suited to, e.g., a mobile robot or drone) at the cost of direct position tracking, with a "centering" spring force letting the operator feel their offset from home. The mode can be toggled back to standard position-matching at any time without disturbing the rest of the control scheme (impedance matching and online tuning are unaffected; only the position-tracking subsystem needs to be disabled while it's active).

## Results

Without any wave-based correction, a standard PD master-slave controller is stable with zero delay but visibly diverges and oscillates into instability once round-trip delay is introduced — this baseline comparison motivates the rest of the thesis. With the full scheme (impedance matching + adjusting the wave command + online tuning) in place, the system stayed stable and passive across every tested combination: free-space motion, contact with a rigid obstacle, both simulated and with added friction, and on the real Novint Falcon hardware, at delays up to 2 seconds round-trip. The velocity-control extension was validated the same way and preserved the same stability guarantees while extending the effective workspace.

## Repository structure

```
Bilateral Telemanipulation under Very Long Delays/
├── README.md
└── docs/
    └── Bilateral_Telemanipulation_under_Very_Long_Delays_Thesis.pdf   full thesis text, derivations, and all experimental results
```

## Tools & technology

Wave-variable / passivity-based control theory, ROS 1 (Noetic Ninjemys, Ubuntu 20.04), `libnifalcon`, Novint Falcon 3-DOF haptic device, simulated mass/friction master-slave modeling.

## Skills demonstrated

Passivity-based control design for time-delayed systems, block-diagram derivation and simplification of a multi-subsystem controller, simulation-to-hardware validation methodology, real-time haptic device integration over ROS, and structured experimental comparison across delay, environment (free space vs. contact), and model fidelity (ideal mass vs. friction vs. real hardware).
