# Mo's

## Mo's with Hilbert Curve Optimization

Reference: <https://codeforces.com/blog/entry/61203>

```rust,noplayground
/// max_n: maximum number of l and r
/// queries: Vec<(id, l, r)>
fn mos_sort(max_n: u32, queries: &[(u32, u32, u32)]) -> Vec<&(u32, u32, u32)> {
    let n_bit = ceil_pow_2(max_n + 1).trailing_zeros();
    let mut arr: Vec<(u64, &(u32, u32, u32))> = queries.iter().map(|q| (0, q)).collect();
    for q in arr.iter_mut() {
        q.0 = hilbert_order(q.1 .1, q.1 .2, n_bit, 0);
    }
    arr.sort_unstable_by_key(|q| q.0);
    arr.into_iter().map(|x| x.1).collect()
}

#[inline(always)]
fn hilbert_order(x: u32, y: u32, pow: u32, rotate: u32) -> u64 {
    if pow == 0 {
        return 0;
    }
    let hpow: u32 = 1 << (pow - 1);
    let mut seg: u32 = if x < hpow {
        if y < hpow {
            0
        } else {
            3
        }
    } else {
        if y < hpow {
            1
        } else {
            2
        }
    };
    seg = (seg + rotate) & 3;

    let (nx, ny) = (x & (x ^ hpow), y & (y ^ hpow));
    let nrot = rotate + ROTATE_DELTA[seg as usize] & 3;
    let sub_square_size = 1u64 << (2 * pow - 2);
    let ans = seg as u64 * sub_square_size;
    let add = hilbert_order(nx, ny, pow - 1, nrot);
    if seg == 1 || seg == 2 {
        ans + add
    } else {
        ans + sub_square_size - add - 1
    }
}

const ROTATE_DELTA: [u32; 4] = [3, 0, 0, 1];

#[inline(always)]
fn ceil_pow_2(y: u32) -> u32 {
    let mut x = y;
    while x != (x & ((!x) + 1)) {
        x -= x & ((!x) + 1);
    }
    if x == y {
        x
    } else {
        x << 1
    }
}
```

## Standard Mo's

```rust,noplayground
/// queries: Vec<(id, l, r)>
fn mos_sort(queries: &mut [(u32, u32, u32)]) {
    let nsq = isqrt(queries.len() as u32);
    queries.sort_unstable_by(|&(_, l1, r1), &(_, l2, r2)| {
        if l1 / nsq == l2 / nsq {
            r1.cmp(&r2)
        } else {
            (l1 / nsq).cmp(&(l2 / nsq))
        }
    });
}

fn isqrt(s: u32) -> u32 {
    let mut x0 = s / 2;
    if x0 != 0 {
        let mut x1 = (x0 + s / x0) / 2;
        while x1 < x0 {
            x0 = x1;
            x1 = (x0 + s / x0) / 2;
        }
        x0
    } else {
        s
    }
}
```