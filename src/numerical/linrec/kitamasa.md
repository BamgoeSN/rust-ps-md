# Kitamasa

Reference: JusticeHui's Blog: <https://justicehui.github.io/hard-algorithm/2021/03/13/kitamasa/>

## Naive Implementation

```rust,noplayground
// Kitamasas
//
// Reference: JusticeHui's Blog: <https://justicehui.github.io/hard-algorithm/2021/03/13/kitamasa/>

fn poly_mul(v: &[u64], w: &[u64], rec: &[u64], m: u64) -> Vec<u64> {
    let mut t = vec![0; 2 * v.len()];

    for j in 0..v.len() {
        for k in 0..w.len() {
            t[j + k] += v[j] * w[k] % m;
            if t[j + k] >= m {
                t[j + k] -= m;
            }
        }
    }

    for j in (v.len()..2 * v.len()).rev() {
        for k in 1..=v.len() {
            t[j - k] += t[j] * rec[k - 1] % m;
            if t[j - k] >= m {
                t[j - k] -= m;
            }
        }
    }

    t[..v.len()].iter().map(|x| *x).collect()
}

/// Finds arr[n] where
/// arr[n+d] = rec[0]arr[n] + rec[1]arr[n+1] + rec[2]arr[n+2] + rec[3]arr[n+3] + ... + rec[d-1]arr[n+d-1]
/// under modulo m where d=rec.len()=arr.len()
fn kitamasa(rec: &[u64], vals: &[u64], mut n: u64, m: u64) -> u64 {
    let recurr: Vec<_> = rec.iter().rev().copied().collect();
    let (mut s, mut t) = (vec![0u64; recurr.len()], vec![0u64; recurr.len()]);
    s[0] = 1;
    if recurr.len() != 1 {
        t[1] = 1;
    } else {
        t[0] = recurr[0];
    }

    while n != 0 {
        if n & 1 != 0 {
            s = poly_mul(&s, &t, &recurr, m);
        }
        t = poly_mul(&t, &t, &recurr, m);
        n >>= 1;
    }

    let mut ret = 0u64;
    for i in 0..recurr.len() {
        ret += s[i] * vals[i] % m;
        if ret >= m {
            ret -= m;
        }
    }
    ret
}
```