# Levels of Relativity

## Motivation 

A point minus a point is a vector, and there are geometry libraries that can statically enforce this by having different vector and point types (eg [euclid](https://docs.rs/euclid/latest/euclid/)), but what about when you want to statically enforce the next subtraction step as well?

With derivatives, the units change:

- $L \longrightarrow m$
- $\frac{dL}{dt} \longrightarrow \frac{m}{s}$
- $\frac{d^2L}{dt^2} \longrightarrow \frac{m}{s^2}$

If you're modelling a position moving with some acceleration, using constant time steps, you'll be dealing with all of these.

```rust

let tick_length_in_seconds: f32 = 0.001;

let initial_position_in_meters: f32 = 5.0;
let initial_velocity_in_meters_per_second: f32 = 1.0;
let acceleration_in_meters_per_second_per_second: f32 = -0.2;

// Units for all of these are in meters
let mut x_m: f32 = initial_position_in_meters;
let mut dx_m: f32 = initial_velocity_in_meters_per_second * tick_length_in_seconds;
let ddx_m: f32 = acceleration_in_meters_per_second_per_second * tick_length_in_seconds * tick_length_in_seconds;

def tick(x: f32, dx: f32, ddx: f32) -> (f32, f32) {
  return (x + dx, dx + dx) // OOPS, TYPO!
  // return (x + dx, dx + ddx)  // This would be correct
}

for x in 0..20 {
  (x_m, dx_m) = tick(x_m, dx_m, ddx_m);
}

print!(x, dx, ddx);

```

In this example, all three of `x`, `dx`, and `ddx` have the same dimension and unit (length, and meters).  Static dimensional analysis (as with the [Units of Measure crate](https://docs.rs/uom/latest/uom/)) would not catch the above typo.

## Solution

These three things are not the same type of thing even if they have the same dimension and unit

- A position
- A velocity times a duration
- An acceleration times a duration squared


A difference between two positions is not a position.  It is a displacement.

A difference between two displacements is not a displacement.  I don't know what it's called, but I can see a pattern here, and that's enough to statically check against.

Google and other search engines haven't helped, so I'mma call it "Level of Relativity" for now.

### Notation

`ThingWithRelativity<ContentsType, Level>(contents) ==> T<Level>(contents)`

- A position of 5 meters would be `T<Level=0>(5m) == T<L=0>(5m) == T<0>(5m)`
- A displacement of 5 meters would be `T<L=1>(5m)`
- Next would be `T<L=2>(5m)`

## Specifics

Ignoring the contents for now.

`ThingWithRelativity<Level=0>(contents) ==> T<L=0>`

Difference makes something more relative.  Like position to displacement

- `T<0> - T<0> == T<1>`
- `T<1> - T<1> == T<2>`
- `T<N> - T<N> == T<N+1>`

You can add or subtract a displacement to a position.  This generalizes

- `T<0> +/- T<1> == T<0>`
- `T<N> +/- T<N+1> == T<0>`

Any other addition or subtraction does not make sense

- `T<N> + T<M>, where N!=M ==> INVALID`
- `T<N> - T<M>, where N!=M and N!=M+1 ==> INVALID`

Bringing back the contents, they just behave like usual

- `T<N>(C1) - T<N>(C2) == T<N+1>(C1-C2)`
- `T<N>(C1) +/- T<N+1>(C2) == T<N>(C1 +/- C2)`

I have no idea what cross-level multiplication or division would look like (what's a position times a displacement?), so I'll just disallow that.

This is kind of analogous to how you can't add two values with different units (what's a meter plus a second?).

- `T<N>(C1) * T<N>(C2) == T<N>(C1 * C2)`


## opt-in behavior

May have another static parameter for allowing opt-in behavior.  eg allowing `ThingWithRelativity<i32, Level=0>::new(5) + 7`

- Assume no relativity implies level of 0, or whatever the other level is (or one higher?)?
- Maybe an enum for default relativity level?  Options: Same, PlusOne, Zero, Disallow

## Running tests

`nix develop`

`cargo test --features trybuild/diff`
