# Maxwell’s Equations

> **Formatting note:** Mathematical expressions use portable Unicode notation, so this file renders correctly without a LaTeX or MathJax extension.

## The key intuition

**Maxwell’s equations are four rules describing how electric charges, electric fields, magnetic fields, and electric currents affect one another.**

Together, they say:

```text
Electric charge
   ↓
creates an electric field

Electric current
   ↓
creates a magnetic field

Changing magnetic field
   ↓
creates a circulating electric field

Changing electric field
   ↓
creates a circulating magnetic field
```

The last two rules form a feedback loop. That feedback allows electric and magnetic fields to travel through empty space as **electromagnetic waves**, including light.

---

## The important objects

**E**

The electric field. It tells a charged particle which direction it would be pushed.

**B**

The magnetic field. It affects moving electric charges.

**ρ**

Electric charge density: how much charge exists near a point.

**J**

Electric current density: how much charge is flowing near a point.

**ε₀**

A constant describing how electric fields behave in empty space.

**μ₀**

A constant describing how magnetic fields behave in empty space.

Two mathematical operations appear repeatedly.

**∇ · field — divergence**

This asks:

> “Is the field spreading outward from this point or converging inward?”

**∇ × field — curl**

This asks:

> “Is the field circulating around this point?”

---

# Equation 1: Electric charges create electric fields

**∇ · E = ρ / ε₀**

This equation is called **Gauss’s law for electricity**.

The equation is trying to say:

> Electric charge is a source or sink of the electric field.

If **ρ is positive**, the electric field tends to spread outward.

If **ρ is negative**, the electric field tends to point inward.

For a tiny example:

```text
Positive charge
      ↑
   ↖  |  ↗
←———  +  ———→
   ↙  |  ↘
      ↓
```

The field spreads away from the positive charge.

In simple terms:

> “Electric field lines begin on positive charges and end on negative charges.”

---

# Equation 2: There are no isolated magnetic charges

**∇ · B = 0**

This equation is called **Gauss’s law for magnetism**.

The equation is trying to say:

> A magnetic field has no isolated starting point or ending point.

Electric fields can begin on positive charges and end on negative charges.

Magnetic fields behave differently. Their field lines form continuous loops.

For a bar magnet:

```text
Outside the magnet:
north → south

Inside the magnet:
south → north

Together:
one closed loop
```

Even when a magnet is cut in half, each piece has both a north and a south pole.

In simple terms:

> “There are no known isolated magnetic north or south charges.”

This is often summarized by saying that magnetic monopoles have not been observed.

---

# Equation 3: A changing magnetic field creates an electric field

**∇ × E = −∂B / ∂t**

This equation is **Faraday’s law of induction**.

The equation is trying to say:

> When a magnetic field changes with time, it creates a circulating electric field.

The right side:

**∂B / ∂t**

means:

> “How quickly is the magnetic field changing?”

The left side:

**∇ × E**

means:

> “How strongly is the electric field circulating?”

The minus sign means the induced electric effect opposes the change that produced it. This is related to **Lenz’s law**.

The flow is:

```text
Magnetic field changes
   ↓
A circulating electric field appears
   ↓
Charges in a wire may begin to move
   ↓
An electric current is generated
```

This is the basic idea behind an electrical generator.

For example:

```text
Move a magnet near a wire coil
   ↓
Magnetic field through the coil changes
   ↓
Electric field circulates around the coil
   ↓
Current flows through the wire
```

In simple terms:

> “Changing magnetism can generate electricity.”

An important subtlety is that the electric field can exist even without a wire. The wire merely gives charges a path along which to move.

---

# Equation 4: Current and changing electric fields create magnetic fields

**∇ × B = μ₀J + μ₀ε₀ ∂E / ∂t**

This is the **Ampère–Maxwell law**.

The equation says that there are two ways to create a circulating magnetic field.

## Part 1: Electric current

**μ₀J**

This says:

> “Flowing electric charge creates a magnetic field.”

Around a straight current-carrying wire, the magnetic field forms circles:

```text
        magnetic field circles
             ↺
             |
             |
current ↑    wire
             |
             |
```

In simple terms:

> “Electricity moving through a wire creates magnetism around the wire.”

## Part 2: A changing electric field

**μ₀ε₀ ∂E / ∂t**

This says:

> “A changing electric field also creates a magnetic field.”

This was Maxwell’s crucial addition to Ampère’s original law.

The full equation says:

```text
Electric current
        or
Changing electric field
   ↓
Circulating magnetic field
```

---

# Why Maxwell added the changing-electric-field term

Consider a capacitor being charged.

A capacitor has two separated metal plates:

```text
wire → | plate     plate | → wire
             gap
```

Current flows through the wire toward the plates.

But charge does not physically cross the empty gap between the plates.

Without Maxwell’s added term, the magnetic field would appear to depend on which surface we use when analyzing the circuit:

```text
Surface crossing the wire:
   sees electric current

Surface passing through the capacitor gap:
   sees no conduction current
```

That would be inconsistent.

While the capacitor charges, however, the electric field in the gap is changing.

```text
Current enters the plates
   ↓
Charge builds up
   ↓
Electric field between the plates changes
   ↓
Changing electric field produces a magnetic field
```

Maxwell’s additional term makes the magnetic-field calculation consistent everywhere.

In simple terms:

> “A changing electric field behaves like a current for the purpose of creating magnetic fields.”

---

# How the four equations fit together

```text
Charge ρ
   ↓ Gauss’s electric law
Electric field E

Current J
   ↓ Ampère–Maxwell law
Magnetic field B

Changing B
   ↓ Faraday’s law
Circulating E

Changing E
   ↓ Ampère–Maxwell law
Circulating B

No magnetic charge
   ↓ Gauss’s magnetic law
Magnetic field lines remain continuous loops
```

The important feedback loop is:

```text
Changing electric field
   ↓
creates magnetic field

Changing magnetic field
   ↓
creates electric field
```

---

# How light emerges

In empty space, suppose there is no local charge and no current:

**ρ = 0**

**J = 0**

Maxwell’s equations then include:

**∇ × E = −∂B / ∂t**

**∇ × B = μ₀ε₀ ∂E / ∂t**

So:

```text
Changing E
   ↓
creates B

Changing B
   ↓
creates E

The changing fields sustain each other
   ↓
The disturbance moves through space
```

The propagation speed predicted by the equations is:

**c = 1 / √(μ₀ε₀)**

This value equals the measured speed of light.

That leads to the major conclusion:

> **Light is an electromagnetic wave.**

Radio waves, microwaves, infrared, visible light, ultraviolet, X-rays, and gamma rays are all the same basic phenomenon at different frequencies.

---

# Differential form versus integral form

The equations above are in **differential form**.

They describe what happens at each point in space:

```text
At this exact location,
is the field spreading or circulating?
```

The integral forms describe what happens over a whole surface or loop:

```text
Across this whole surface,
how much field passes through?

Around this whole loop,
how much does the field circulate?
```

For example, Gauss’s electric law can also be written conceptually as:

**Electric flux through a closed surface = enclosed charge / ε₀**

In simple terms:

> “The total electric field leaving a closed region depends on how much charge is inside.”

The differential and integral forms express the same physical laws at different scales.

---

# What Maxwell’s equations do not describe by themselves

Maxwell’s equations describe how electromagnetic fields are generated and evolve.

To determine how a charged particle moves, we also need the **Lorentz force law**:

**F = q(E + v × B)**

It says:

```text
Electric field E
   ↓
pushes a charge directly

Magnetic field B
   ↓
pushes a moving charge sideways
```

So the full interaction is:

```text
Charges and currents
   ↓
create E and B through Maxwell’s equations

E and B
   ↓
push charges through the Lorentz force

Moving charges
   ↓
change the fields again
```

---

# Bottom line

**Maxwell’s equations say that charges create electric fields, currents create magnetic fields, and changing electric and magnetic fields create each other.**

That single framework unifies:

```text
electricity
magnetism
induction
radio waves
and light
```
