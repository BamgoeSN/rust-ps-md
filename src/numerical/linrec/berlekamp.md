# Berlekamp-Massey

```rust,noplayground
fn rem_pow(mut base: i64, mut exp: i64, m: i64) -> i64 {
    let mut result = 1;
    while exp != 0 {
        if exp & 1 != 0 {
            result = (result * base) % m;
        }
        exp >>= 1;
        base = (base * base) % m;
    }
    result
}

/// Finds rec[n] which satisfies
/// vals[d] = rec[0]vals[0] + rec[1]vals[1] + ... + rev[d-1]vals[d-1]
/// with minimum n.
pub fn berlekamp_massey(vals: &[u64], m: u64) -> Vec<u64> {
    let m = m as i64;
    let mut cur: Vec<i64> = Vec::new();
    let (mut lf, mut ld) = (0, 0);
    let mut ls: Vec<i64> = Vec::new();
    for i in 0..vals.len() {
        let mut t = 0;
        for (j, v) in cur.iter().enumerate() {
            t = (t + vals[i - j - 1] as i64 * v) % m;
        }

        if (t - vals[i] as i64) % m == 0 {
            continue;
        }

        if cur.len() == 0 {
            cur = vec![0; i + 1];
            lf = i;
            ld = (t - vals[i] as i64) % m;
            continue;
        }

        let k = -(vals[i] as i64 - t) * rem_pow(ld, m - 2, m) % m;
        let mut c: Vec<i64> = vec![0; i - lf + ls.len()];
        c[i - lf - 1] = k as i64;
        for (p, j) in ls.iter().enumerate() {
            c[i - lf + p] = -j * k % m;
        }

        if c.len() < cur.len() {
            c.extend((0..(cur.len() - c.len())).map(|_| 0));
        }

        for j in 0..cur.len() {
            c[j] = (c[j] + cur[j]) % m;
        }

        if i - lf + ls.len() >= cur.len() {
            ls = cur;
            lf = i;
            ld = (t - vals[i] as i64) % m;
        }

        cur = c;
    }

    for i in 0..cur.len() {
        cur[i] = (cur[i] % m + m) % m;
    }

    cur.into_iter().rev().map(|x| x as u64).collect()
}
```