# Longest Increasing Subsequence

## Length
```rust,noplayground
fn lis_len(arr: &[i64]) -> usize {
    let mut table: Vec<i64> = vec![arr[0]];
    for &v in arr[1..].iter() {
        let p = table.partition_point(|&x| x < v);
        if p == table.len() {
            table.push(v);
        } else {
            table[p] = v;
        }
    }
    table.len()
}
```

## Sequence
```rust,noplayground
fn lis(arr: &[i64]) -> Vec<i64> {
    let n = arr.len();
    let mut seq: Vec<i64> = Vec::with_capacity(n + 1);
    seq.push(i64::MIN);
    seq.extend(arr.iter().copied());

    let mut back = vec![0usize; n + 1];
    let mut table = vec![0usize];

    for (i, &v) in seq.iter().enumerate().skip(1) {
        let p = table.partition_point(|&x| seq[x] < v);
        if p == table.len() {
            table.push(i);
        } else {
            table[p] = i;
        }
        back[i] = table[p - 1];
    }

    let mut ptr = *table.last().unwrap();
    let mut ans: Vec<i64> = Vec::with_capacity(table.len() - 1);
    while ptr != 0 {
        ans.push(seq[ptr]);
        ptr = back[ptr];
    }

    ans.reverse();
    ans
}
```