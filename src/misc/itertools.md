# Iterator Tools

## Grid Iteration
`gen_diriter(r, c, tr, tc)` returns an iterator of `(usize, usize)` which iterates through right, down, left, up from `(r, c)` in a grid of `tr` rows and `tc` columns. `gen_torusiter(r, c, tr, tc)` acts similarly, but in a grid where your position wrap up when you go out of bounds.

The direction of iteration can be customized by modifying `DR` and `DC`.

```rust,noplayground
const DR: [usize; 4] = [0, 1, 0, !0];
const DC: [usize; 4] = [1, 0, !0, 0];

fn gen_diriter(r: usize, c: usize, tr: usize, tc: usize) -> impl Iterator<Item = (usize, usize)> {
    std::iter::zip(DR.iter(), DC.iter())
        .map(move |(&dr, &dc)| (r.wrapping_add(dr), c.wrapping_add(dc)))
        .filter(move |&(nr, nc)| nr < tr && nc < tc)
}

fn gen_torusiter(r: usize, c: usize, tr: usize, tc: usize) -> impl Iterator<Item = (usize, usize)> {
    std::iter::zip(DR.iter(), DC.iter())
        .map(move |(&dr, &dc)| (r.wrapping_add(dr), c.wrapping_add(dc)))
        .map(move |(nr, nc)| {
            let r = if nr > usize::MAX / 2 {
                let delta = (usize::MAX - nr) % tr;
                tr - 1 - delta
            } else {
                nr
            };
            let c = if nc > usize::MAX / 2 {
                let delta = (usize::MAX - nc) % tc;
                tc - 1 - delta
            } else {
                nc % tc
            };
            (r, c)
        })
}
```

## Cartesian Product
`cart_prod(a, b)` returns an iterator of cartesian product of two iterators `a` and `b`.

```rust,noplayground
fn cart_prod<I, J, S, T>(a: I, b: J) -> impl Iterator<Item = (S, T)>
where I: Iterator<Item = S>, J: Iterator<Item = T> + Clone, S: Clone {
    a.flat_map(move |a| b.clone().map(move |b| (a.clone(), b)))
}
```

## Intersperse
`intersperse(iter, v)` returns an iterator which inserts `v` between elements of `iter`.

```rust,noplayground
fn intersperse<T: Clone>(iter: impl Iterator<Item = T>, with: T) -> impl Iterator<Item = T> {
    iter.map(move |v| [with.clone(), v]).flatten().skip(1)
}
```