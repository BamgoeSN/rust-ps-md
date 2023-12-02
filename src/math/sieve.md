# Sieve

Sieve algorithms find all prime numbers below a specified integer. Additionally, they efficiently compute values of multiplicative functions for all integers up to that specified limit.

## Finding primes
`sieve` returns a vector containing all prime numbers that are less than or equal to `max_val`.

This function runs with a time complexity of \\(O(N)\\) where \\(N\\) is the value of `max_val`. This efficiency is achieved with linear sieve.
This function is further optimized by excluding multiples of \\(2\\) and \\(3\\) in advance.

```rust,noplayground
fn sieve(max_val: usize) -> Vec<usize> {
	let mut primes = vec![2, 3];
	let mut is_prime = vec![true; max_val / 3 + 1];

	for i in 0..is_prime.len() {
		let j = 6 * (i >> 1) + 5 + ((i & 1) << 1);
		if is_prime[i] {
			primes.push(j);
		}
		for &p in primes[2..].iter() {
			let v = j * p;
			if v > max_val {
				break;
			}
			is_prime[v / 3 - 1] = false;
			if j % p == 0 {
				break;
			}
		}
	}

	primes
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

## With Möbius Function
```rust,noplayground
fn mobius_sieve(max_val: usize) -> (Vec<i8>, Vec<usize>) {
    let mut primes = vec![];
    let mut mu = vec![2i8; max_val + 1];
    (mu[0], mu[1]) = (0, 1);

    for i in 2..=max_val {
        if mu[i] == 2 {
            primes.push(i);
            mu[i] = -1;
        }
        for &p in primes.iter() {
            let v = i * p;
            if v > max_val {
                break;
            }
            if i % p == 0 {
                mu[v] = 0;
                break;
            } else {
                mu[v] = -mu[i];
            }
        }
    }

    (mu, primes)
}
```

---

Last modified on 231203.