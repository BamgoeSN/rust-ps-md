# Bostan-Mori
Reference - [Alin Bostan, Ryuhei Mori. A Simple and Fast Algorithm for Computing the N-th Term of a Linearly Recurrent Sequence. SOSA’21 (SIAM Symposium on Simplicity in Algorithms), Jan 2021, Alexandria, United States. ffhal-02917827v2f](https://hal.inria.fr/hal-02917827v2/document)

## Calculating one value of the sequence

### Naive multiplication, under prime modulo
```rust,noplayground
/// Returns g, s, t s.t. g=gcd(a,b) and as+bt=r
#[inline(always)]
fn ext_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    let (mut s, mut old_s) = (0, 1);
    let (mut r, mut old_r) = (b, a);
    while r != 0 {
        let q = old_r / r;

        let new_r = old_r - q * r;
        old_r = r;
        r = new_r;

        let new_s = old_s - q * s;
        old_s = s;
        s = new_s;
    }

    (
        old_r,
        old_s,
        if b != 0 { (old_r - old_s * a) / b } else { 0 },
    )
}

fn mod_inv(a: u64, m: u64) -> u64 {
    let (_, mut x, _) = ext_gcd((a % m) as i64, m as i64);
    if x < 0 {
        x += m as i64;
    }
    x as u64 % m
}

/// f = f[0] + f[1]x + f[2]x2 + ...
#[inline(always)]
fn poly_mul(f: &[u64], g: &[u64], m: u64) -> Vec<u64> {
    let mut h: Vec<u64> = vec![0; f.len() + g.len() - 1];
    for (i, &u) in f.iter().enumerate() {
        // u*x**i
        for (j, &v) in g.iter().enumerate() {
            // v*x**j
            h[i + j] += u * v;
            h[i + j] %= m;
        }
    }
    while let Some(&v) = h.last() {
        if v != 0 {
            break;
        }
        h.pop();
    }
    h
}

#[inline(always)]
fn poly_mul_even_order(f: &[u64], g: &[u64], m: u64) -> Vec<u64> {
    let mut h: Vec<u64> = vec![0; (f.len() + g.len()) / 2 + 2];
    for (i, &u) in f.iter().enumerate() {
        if i & 1 == 0 {
            for (j, &v) in g.iter().enumerate().step_by(2) {
                h[(i + j) >> 1] += u * v;
                h[(i + j) >> 1] %= m;
            }
        } else {
            for (j, &v) in g.iter().enumerate().skip(1).step_by(2) {
                h[(i + j) >> 1] += u * v;
                h[(i + j) >> 1] %= m;
            }
        }
    }
    while let Some(&v) = h.last() {
        if v != 0 {
            break;
        }
        h.pop();
    }
    h
}

#[inline(always)]
fn poly_mul_odd_order(f: &[u64], g: &[u64], m: u64) -> Vec<u64> {
    let mut h: Vec<u64> = vec![0; (f.len() + g.len()) / 2 + 2];
    for (i, &u) in f.iter().enumerate() {
        if i & 1 != 0 {
            for (j, &v) in g.iter().enumerate().step_by(2) {
                h[(i + j) >> 1] += u * v;
                h[(i + j) >> 1] %= m;
            }
        } else {
            for (j, &v) in g.iter().enumerate().skip(1).step_by(2) {
                h[(i + j) >> 1] += u * v;
                h[(i + j) >> 1] %= m;
            }
        }
    }
    while let Some(&v) = h.last() {
        if v != 0 {
            break;
        }
        h.pop();
    }
    h
}

/// f(x) -> f(-x)
#[inline(always)]
fn get_neg_x(f: &[u64], m: u64) -> Vec<u64> {
    f.iter()
        .enumerate()
        .map(|(i, &v)| if i & 1 == 0 { v } else { m - v })
        .collect()
}

/// Finds arr[n] where
/// arr[n+d] = rec[0]arr[n] + rec[1]arr[n+1] + rec[2]arr[n+2] + rec[3]arr[n+3] + ... + rec[d-1]arr[n+d-1]
/// under modulo m where d=rec.len()=arr.len()
fn bostan_mori(rec: &[u64], vals: &[u64], n: u64, m: u64) -> u64 {
    if vals.len() as u64 > n {
        return vals[n as usize];
    }
    let d = rec.len();

    let mut q: Vec<u64> = Vec::with_capacity(d + 1);
    q.push(1);
    q.extend(rec.iter().map(|&v| m - v).rev());

    let mut p = poly_mul(vals, &q, m);
    p.truncate(d);

    let mut n = n;
    while n >= 1 {
        let mq = get_neg_x(&q, m);
        if n & 1 == 0 {
            p = poly_mul_even_order(&p, &mq, m);
        } else {
            p = poly_mul_odd_order(&p, &mq, m);
        }
        q = poly_mul_even_order(&q, &mq, m);
        n >>= 1;
    }
    p[0] * mod_inv(q[0], m) // Requires GCD(q[0], m) = 1, it's safe to have m as a prime
}
```