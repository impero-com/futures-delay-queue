# futures-delay-queue

A queue of delayed elements running backed by [async-std](https://github.com/async-rs/async-std) and [futures-timer](https://github.com/async-rs/futures-timer).

Once an element is inserted into the `DelayQueue`, it is yielded once the
specified deadline has been reached.

The delayed items can be consumed through a channel returned at creation.

## Implementation

The delays are spawned and a timeout races against a oneshot channel that can be
triggered with the [`DelayHandle`]. If the timeout occurs before cancelation the
item is yielded through the receiver channel.

## Usage

Elements are inserted into `DelayQueue` using the [`insert`] or
[`insert_at`] methods. A deadline is provided with the item and a [`DelayHandle`] is
returned. The delay handle is used to remove the entry.

The delays can be canceled by calling the [`cancel`] method on the [`DelayHandle`].

## Example

```rust
use futures_delay_queue::delay_queue;
use std::time::Duration;

#[async_std::main]
async fn main() {
    let (dq, rx) = delay_queue::<i32>(3);

    let _ = dq.insert(1, Duration::from_secs(2));
    let d = dq.insert(2, Duration::from_secs(1));
    d.cancel();
    let _ = dq.insert(3, Duration::from_secs(3));

    assert_eq!(rx.recv().await, Some(1));
    assert_eq!(rx.recv().await, Some(3));

    drop(dq);
    assert_eq!(rx.recv().await, None);
}
```

[`insert`]: #method.insert
[`insert_at`]: #method.insert_at
[`cancel`]: #method.cancel
[`DelayHandle`]: struct.DelayHandle.html

License: Apache-2.0/MIT
