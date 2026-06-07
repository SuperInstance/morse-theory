# morse-theory

> **Read the shape of a manifold through its critical points.**

[![crates.io](https://img.shields.io/crates/v/morse-theory.svg)](https://crates.io/crates/morse-theory)
[![docs.rs](https://docs.rs/morse-theory/badge.svg)](https://docs.rs/morse-theory)
[![license](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![tests](https://img.shields.io/badge/tests-14-passing-green.svg)]()

Morse theory reveals the topology of manifolds through the critical points of smooth functions. Critical points of index k attach k-dimensional handles, and the Morse inequalities bound the Betti numbers.

---

## Why This Exists

Morse theory is one of the deepest connections between **analysis** (smooth functions) and **topology** (homology). It tells you:

1. How many holes a manifold has (Morse inequalities)
2. How to build the manifold from simple pieces (handle attachments)
3. The Euler characteristic from a single function (χ = Σ(-1)^k μ_k)

This library makes Morse theory **computational** — define a Morse function on your manifold, and we compute the rest.

---

## Architecture

```
    ┌───────────────────┐
    │  Morse Function f │
    │  on manifold M    │
    └────────┬──────────┘
             │
    ┌────────▼──────────┐      ┌──────────────────┐
    │  Critical Points  │─────►│  Morse Inequalities│
    │  (index, value)   │      │  μ_k ≥ β_k       │
    └────────┬──────────┘      └──────────────────┘
             │
    ┌────────▼──────────┐      ┌──────────────────┐
    │  Morse Complex    │─────►│  Homology        │
    │  C_k = <CP of     │      │  H_k(M) ≅ ker/  │
    │   index k>        │      │       im         │
    └────────┬──────────┘      └──────────────────┘
             │
    ┌────────▼──────────┐
    │  Handle           │
    │  Attachment       │
    │  M = ∪ h_k        │
    └───────────────────┘
```

---

## Installation

```toml
[dependencies]
morse-theory = "0.1.0"
```

---

## Quick Start

```rust
use morse_theory::{MorseFunction, CriticalPoint};

// Height function on the torus T²
let mut f = MorseFunction::new("height on torus", 2);
f.add_critical_point(CriticalPoint::new(0, 0, 0.0));  // minimum
f.add_critical_point(CriticalPoint::new(1, 1, 1.0));  // saddle
f.add_critical_point(CriticalPoint::new(2, 1, 2.0));  // saddle
f.add_critical_point(CriticalPoint::new(3, 2, 3.0));  // maximum

// Morse numbers: μ₀=1, μ₁=2, μ₂=1
assert_eq!(f.morse_number(0), 1);
assert_eq!(f.morse_number(1), 2);
assert_eq!(f.morse_number(2), 1);

// Euler characteristic: χ(T²) = 1 - 2 + 1 = 0
assert_eq!(f.euler_characteristic(), 0);
```

---

## Usage Examples

### Example 1: Morse Inequalities

```rust
use morse_theory::MorseInequalities;

// Torus: μ = [1, 2, 1], β = [1, 2, 1]
let morse = vec![1, 2, 1];
let betti = vec![1, 2, 1];

// Weak: μ_k ≥ β_k for all k
let weak = MorseInequalities::weak_inequality(&morse, &betti);
assert!(weak.iter().all(|(_, _, s)| *s));

// Strong: alternating sum inequality
assert!(MorseInequalities::strong_inequality(&morse, &betti));
```

### Example 2: Sphere S²

```rust
use morse_theory::{MorseFunction, CriticalPoint};

let mut f = MorseFunction::new("height on S²", 2);
f.add_critical_point(CriticalPoint::new(0, 0, -1.0)); // south pole
f.add_critical_point(CriticalPoint::new(1, 1, 0.0));  // equator
f.add_critical_point(CriticalPoint::new(2, 2, 1.0));  // north pole

// χ(S²) = 1 - 1 + 1 = 1
assert_eq!(f.euler_characteristic(), 1);
```

### Example 3: Handle Attachment

```rust
use morse_theory::{build_manifold, CriticalPoint};

let cps = vec![
    CriticalPoint::new(0, 0, 0.0),
    CriticalPoint::new(1, 1, 2.0),
    CriticalPoint::new(2, 0, 1.0),  // another minimum
];

// Handles sorted by function value
let handles = build_manifold(&cps, 2);
assert_eq!(handles.len(), 3);
assert_eq!(handles[0].attached_at_value, 0.0); // sorted
```

### Example 4: Morse Complex

```rust
use morse_theory::{MorseFunction, CriticalPoint, MorseComplex};

let mut f = MorseFunction::new("f", 2);
f.add_critical_point(CriticalPoint::new(0, 0, 0.0));
f.add_critical_point(CriticalPoint::new(1, 1, 1.0));
f.add_critical_point(CriticalPoint::new(2, 1, 2.0));
f.add_critical_point(CriticalPoint::new(3, 2, 3.0));

let complex = MorseComplex::from_morse_function(&f);
assert_eq!(complex.rank(0), 1); // one minimum
assert_eq!(complex.rank(1), 2); // two saddles
assert_eq!(complex.rank(2), 1); // one maximum
assert_eq!(complex.euler_characteristic(), 0);
```

---

## Mathematical Background

### Morse Functions

A smooth function f: M → ℝ is **Morse** if all critical points are non-degenerate (Hessian has full rank). At each critical point p:

- **Index** k = number of negative eigenvalues of the Hessian
- Critical point of index k ↔ attach a k-handle to the manifold

### Morse Inequalities

Let μ_k = number of critical points of index k, β_k = k-th Betti number.

**Weak**: μ_k ≥ β_k for all k

**Strong**: Σ_{j≤k} (-1)^{k-j} μ_j ≥ Σ_{j≤k} (-1)^{k-j} β_j

**Equality** (Euler): Σ(-1)^k μ_k = Σ(-1)^k β_k = χ(M)

### The Torus Example

Height function on T² has 4 critical points:
- 1 minimum (index 0)
- 2 saddles (index 1)
- 1 maximum (index 2)

μ₀=1, μ₁=2, μ₂=1 → β₀=1, β₁=2, β₂=1 → χ = 0 ✓

---

## API Reference

| Type | Description |
|------|-------------|
| `CriticalPoint` | Non-degenerate critical point with index, value, position |
| `MorseFunction` | Morse function with critical points, Morse numbers, Euler characteristic |
| `MorseInequalities` | Weak and strong Morse inequality checking |
| `MorseComplex` | Chain complex from gradient flow lines |
| `HandleAttachment` | Handle decomposition of manifolds |

---

## Performance

```
test result: ok. 14 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Zero dependencies. Stack-allocated critical points. O(n log n) handle sorting.

---

## References

- Milnor, J. *Morse Theory* (1963)
- Milnor, J. *Lectures on the h-Cobordism Theorem* (1965)
- Matsumoto, Y. *An Introduction to Morse Theory* (2002)

---

## License

MIT © [SuperInstance](https://github.com/SuperInstance)

---

*Part of the [Exocortex](https://github.com/SuperInstance/exocortex) project.*
