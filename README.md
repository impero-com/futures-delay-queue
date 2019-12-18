# Asynchronous delay queue

<!-- cargo-sync-readme start -->

A queue of delayed elements backed by [futures-timer](https://crates.io/crates/futures-timer) that can be used with both
- [async-std](https://crates.io/crates/async-std) as default, and
- [tokio](https://crates.io/crates/tokio) with feature "use-tokio"

An element is inserted into the [`DelayQueue`] and will be yielded once the
specified deadline has been reached.

The delayed items can be consumed through a channel returned at creation.

# Implementation

The delays are spawned and a timeout races against a reset channel that can be
triggered with the [`DelayHandle`]. If the timeout occurs before cancelation
or a reset the item is yielded through the receiver channel.

# Usage

Elements are inserted into [`DelayQueue`] using the [`DelayQueue::insert`] or
[`DelayQueue::insert_at`] methods. A deadline is provided with the item and a [`DelayHandle`] is
returned. The delay handle is used to remove the entry.

The delays can be configured with the [`DelayHandle::reset_at`] or the [`DelayHandle::reset`] method or canceled by
calling the [`DelayHandle::cancel`] method. Dropping the handle will not cancel the delay.

Modification of the delay fails if the delayed item expired in the meantime. In this case
an [`ErrorAlreadyExpired`] will be returned. If modification succeeds the handle will
be returned back to the caller.

# Example

```rust
use futures_delay_queue::delay_queue;
use std::time::Duration;

#[async_std::main]
async fn main() {
    let (delay_queue, rx) = delay_queue::<i32>(3);

    let delay_handle = delay_queue.insert(1, Duration::from_millis(20));
    assert!(delay_handle.reset(Duration::from_millis(40)).await.is_ok());

    let delay_handle = delay_queue.insert(2, Duration::from_millis(10));
    assert!(delay_handle.cancel().await.is_ok());

    let delay_handle = delay_queue.insert(3, Duration::from_millis(30));

    assert_eq!(rx.receive().await, Some(3));
    assert_eq!(rx.receive().await, Some(1));

    drop(delay_queue);
    assert_eq!(rx.receive().await, None);
}
```

[`insert`]: #method.insert
[`insert_at`]: #method.insert_at
[`cancel`]: #method.cancel
[`DelayHandle`]: struct.DelayHandle.html

<!-- cargo-sync-readme end -->