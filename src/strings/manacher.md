# Manacher

For an array \\(A\\) of length \\(n\\), `manacher(A)` returns a vector \\(M\\) where, for every \\(i \in \left[0, n\right)\\), \\(A_{i-j} = A_{i+j}\\) holds for every \\(j \in \left[0, M_i \right)\\).

```rust,noplayground
fn manacher<T: Eq>(arr: &[T]) -> Vec<u32> {
    let n = arr.len() as u32;
    let mut mana: Vec<u32> = vec![1; n as usize];
    let mut r: u32 = 1;
    let mut p: u32 = 0;

    for i in 1..arr.len() as u32 {
        if i + 1 >= r {
            mana[i as usize] = 1;
        } else {
            let j = 2 * p - i;
            mana[i as usize] = mana[j as usize].min(r - i);
        }

        while mana[i as usize] <= i && i + mana[i as usize] < n {
            if arr[(i - mana[i as usize]) as usize] != arr[(i + mana[i as usize]) as usize] {
                break;
            }
            mana[i as usize] += 1;
        }

        if r < mana[i as usize] + i {
            r = mana[i as usize] + i;
            p = i;
        }
    }

    mana
}
```