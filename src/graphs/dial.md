# Dial
Dial algorithm is an alternative to Dijkstra algorithm, which can be used when the maximum value of the edge costs is small. Instead of using a heap, dial algorithm uses a queue of vectors to sort searched paths by their distances. Generally the performance of it is quite similar to that of Dijkstra, but if somehow you want to use this instead, then go ahead!

The usage is exactly the same with Dijkstra algorithm, except that the name of the function is different.

## Code
```rust,noplayground
trait HasNz {
    type NzType;
    fn into_nz(self) -> Option<Self::NzType>;
    fn retrieve(nz: Self::NzType) -> Self;
}

macro_rules! impl_hasnz {
    ($($t:ty, $n:ty);*) => { $(
        impl HasNz for $t {
            type NzType = $n;
            fn into_nz(self) -> Option<$n> { <$n>::new(self) }
            fn retrieve(nz: $n) -> Self { nz.get() }
        }
    )* };
}

impl_hasnz!(i8, NonZeroI8; i16, NonZeroI16; i32, NonZeroI32; i64, NonZeroI64; i128, NonZeroI128; isize, NonZeroIsize);
impl_hasnz!(u8, NonZeroU8; u16, NonZeroU16; u32, NonZeroU32; u64, NonZeroU64; u128, NonZeroU128; usize, NonZeroUsize);

fn dial<T>(graph: &[Vec<(usize, T)>], src: usize) -> Vec<Option<T>>
where
    T: Copy + From<u8> + Into<u32> + Add<Output = T> + Sub<Output = T> + Eq + Ord + HasNz,
    <T as HasNz>::NzType: Copy,
{
    let max_cost: u32 = graph.iter().map(|list| list.iter().map(|&(_, v)| v.into())).flatten().max().unwrap_or(0);

    let mut dist: Vec<Option<T::NzType>> = vec![None; graph.len()];
    dist[src] = {
        let one: T = 1.into();
        one.into_nz()
    };

    let mut qcnt: usize = 1;
    let mut queue = std::collections::VecDeque::with_capacity(max_cost as usize + 2);
    for _ in 0..=queue.capacity() {
        queue.push_back(vec![]);
    }
    queue[0].push(src as u32);

    let mut curr_cost: T = 0.into();
    while qcnt != 0 {
        curr_cost = curr_cost + 1.into();
        let mut hand = queue.pop_front().unwrap();

        while let Some(curr) = hand.pop() {
            qcnt -= 1;
            if dist[curr as usize].map_or(false, |x| T::retrieve(x) < curr_cost) {
                continue;
            }

            for &(next, weight) in graph[curr as usize].iter() {
                let next_cost = curr_cost + weight;

                if dist[next].map_or(true, |x| T::retrieve(x) > next_cost) {
                    dist[next] = next_cost.into_nz();

                    qcnt += 1;
                    if weight == 0.into() {
                        hand.push(next as u32);
                    } else {
                        queue[weight.into() as usize - 1].push(next as u32);
                    }
                }
            }
        }

        queue.push_back(hand);
    }

    dist.iter().map(|x| x.map(|x| T::retrieve(x) - 1.into())).collect()
}
```
