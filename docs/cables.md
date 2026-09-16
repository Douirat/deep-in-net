# Ethernet Cables — Straight-Through vs Crossover

## 1. What is a Pin?

A **pin** is an electrical contact inside an Ethernet (RJ-45) connector.

An Ethernet connector has **8 pins**:

```text
1 2 3 4 5 6 7 8
• • • • • • • •
```

Each pin connects to a wire inside the cable.

---

## 2. The Important Pins

For traditional **10/100 Mbps Ethernet**:

```text
Pins 1, 2 → one signal pair
Pins 3, 6 → another signal pair
```

The important concept is **TX (Transmit)** and **RX (Receive)**.

### PC / Router — MDI

```text
Pins 1,2 → TX
Pins 3,6 → RX
```

### Switch — MDIX

```text
Pins 1,2 → RX
Pins 3,6 → TX
```

Therefore, PC and switch have **opposite TX/RX arrangements**.

---

# 3. Straight-Through Cable

A straight-through cable connects the **same pin number to the same pin number**:

```text
1 ───────── 1
2 ───────── 2
3 ───────── 3
6 ───────── 6
```

Because a PC and switch have opposite TX/RX arrangements:

```text
PC                         Switch

TX (1,2) ───────────────> RX (1,2)

RX (3,6) <─────────────── TX (3,6)
```

Therefore:

> **Different/opposite TX/RX arrangements → straight-through**

### Common examples

```text
PC     → Switch    = Straight
PC     → Router    = Straight
Router → Switch    = Straight
```

---

# 4. Crossover Cable

A crossover cable changes the pin connections:

```text
1 ───────── 3
2 ───────── 6

3 ───────── 1
6 ───────── 2
```

This is necessary when both devices have the **same TX/RX arrangement**.

For example:

```text
PC                         PC

TX (1,2) ───────────────> TX (1,2) ❌
RX (3,6) <─────────────── RX (3,6) ❌
```

Without crossing, TX would connect to TX and RX to RX.

The crossover fixes it:

```text
PC                         PC

TX (1,2) ───────────────> RX (3,6)
RX (3,6) <─────────────── TX (1,2)
```

Therefore:

> **Same TX/RX arrangement → crossover**

### Common traditional examples

```text
PC     → PC       = Crossover
Switch → Switch   = Crossover
Router → Router   = Crossover
```

---

# 5. The Rule to Remember

Do **not** memorize only:

> PC → Switch = straight

Understand the reason:

```text
Different / opposite TX-RX arrangement
                ↓
         STRAIGHT-THROUGH
```

```text
Same TX-RX arrangement
                ↓
            CROSSOVER
```

The purpose of the cable is always:

> **TX must reach RX.**

---

# 6. Quick Table

| Devices         | Traditional Cable | Reason                      |
| --------------- | ----------------- | --------------------------- |
| PC → Switch     | Straight-through  | Opposite TX/RX arrangements |
| PC → Router     | Straight-through  | Opposite TX/RX arrangements |
| Router → Switch | Straight-through  | Opposite TX/RX arrangements |
| PC → PC         | Crossover         | Same TX/RX arrangement      |
| Switch → Switch | Crossover         | Same TX/RX arrangement      |
| Router → Router | Crossover         | Same TX/RX arrangement      |

---

# 7. Modern Ethernet: Auto-MDIX

Modern Ethernet interfaces often support **Auto-MDIX**.

The device can automatically detect that TX/RX are connected incorrectly and internally adjust the interface.

Therefore, in modern real-world networks:

```text
Straight-through or crossover
          ↓
   Often both work
```

But in **Cisco/Packet Tracer exercises**, learn the traditional rule unless the exercise specifies Auto-MDIX.

---

# 8. Mental Model

When choosing an Ethernet cable, ask:

1. **What type of connection is this?**
2. **What are the TX/RX arrangements of the two ports?**
3. **Are they opposite?**

   * Yes → **Straight-through**
4. **Are they the same?**

   * Yes → **Crossover**

### The fundamental idea

```text
STRAIGHT:

TX ───────────────> RX
RX <─────────────── TX
```

```text
CROSSOVER:

TX ───────────────> RX
       ↘     ↙
RX <─────────────── TX
```

**The cable type is determined by how the two interfaces need their TX and RX paths connected.**
