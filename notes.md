## generic over `Result<>` return type

the try keyword must consider `panic` or `!panic`

## `.do` keyword

```rust
fn compute<effect<KernelSpace | UserSpace | PreComputed>>() -> try Response
where
    effect: try<ErrType>,
    effect<KernelSpace>: !alloc + !panic + !async,
    effect<UserSpace>: alloc + panic + ?async,
    effect<PreComputed>: const
{
    if effect<KernelSpace> {
        // ensures that in "KernelSpace" will not alloc, panic or run futures
    }
    if effect<UserSpace> {
        // allow allocations and futures
    }
    if effect<PreComputed> {
        // only compile-time evaluation
    }
}

fn caller()
where
    effect: alloc + panic + async
{
    // allowed
    let res: Response = compute<effect<UserSpace>>().do<await + try>;
    let res: Result<_, ErrType> = compute<effect<UserSpace>>().do<await + !try>;
    let res: impl Future<Output = Result<_, ErrType>> =
        compute<effect<UserSpace>>().do<!await + !try>;
    let res: impl Future<Output = Response> =
        compute<effect<UserSpace>>().do<!await + try>;
}
```
