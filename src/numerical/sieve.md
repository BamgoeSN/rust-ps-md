# Sieve

Linear sieve can find primes below \\(N\\) in \\(O(N)\\) time. With searching for primes, it can also calculate values of multiplicative functions for values below \\(N\\) in linear time.

## Finding primes
```rust,noplayground
fn sieve(max_val: usize) -> (Vec<bool>, Vec<usize>) {
    let mut primes = vec![];
    let mut is_prime = vec![true; max_val + 1];
    is_prime[0] = false;
    is_prime[1] = false;

    for i in 2..=max_val {
        if is_prime[i] {
            primes.push(i);
        }
        for &p in primes.iter() {
            let v = i * p;
            if v > max_val {
                break;
            }
            is_prime[v] = false;
            if i % p == 0 {
                break;
            }
        }
    }

    (is_prime, primes)
}
```

## With Euler Phi Function
```rust,noplayground
fn phi_sieve(max_val: usize) -> (Vec<bool>, Vec<usize>, Vec<usize>) {
    let mut primes = vec![];
    let mut is_prime = vec![true; max_val + 1];
    is_prime[0] = false;
    is_prime[1] = false;
    let mut phi = vec![0; max_val + 1];

    for i in 2..=max_val {
        if is_prime[i] {
            primes.push(i);
            phi[i] = i - 1;
        }
        for &p in primes.iter() {
            let v = i * p;
            if v > max_val {
                break;
            }
            is_prime[v] = false;
            if i % p == 0 {
                phi[v] = phi[i] * p;
                break;
            } else {
                phi[v] = phi[i] * phi[p]
            }
        }
    }

    (is_prime, phi, primes)
}
```
