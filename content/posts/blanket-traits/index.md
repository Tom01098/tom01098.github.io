+++ 
draft = false
date = 2026-02-18
title = "Deduplicate trait bounds with blanket traits"
+++

I've recently learned about blanket traits, a slightly hidden feature of Rust's
excellent trait system. It gave me the single biggest 'aha!' moment that I've had 
for writing idiomatic Rust code, but I've seen little discussion about it. In short,
they let you implement a trait for all types that meet some conditions.
Let's jump into a toy example that should make why I like it clear!

I'm writing the world's most pointless web server. It listens for TCP requests and
responds to them with the bytes it received. Let's look at the core function which
does this.

```rust
use std::io::{Read, Write};
use std::net::TcpStream;

fn reflect_stream(stream: &mut TcpStream) -> std::io::Result<()> {
    let mut buffer = [0; 512];

    loop {
        let bytes_read = stream.read(&mut buffer)?;
        if bytes_read == 0 {
            return Ok(());  // connection closed.
        }
        stream.write_all(&buffer[0..bytes_read])?;
    }
}
```

We're responsible programmers (right??), so we should write an automated test 
for this function. But that's where problems start to show up. Testing a `TcpStream`
is quite hard because it's coupled tightly to I/O, which never plays nicely in unit
tests. In other languages, I might consider creating a mock object that looks like
a `TcpStream` (for my Python besties, I would use a `MagicMock`).

But let's take a step back – what is the bare-minimum we need `stream` to do? We need 
to be able to read bytes from it and write bytes to it. Conveniently, this is exactly
what the `Read` and `Write` traits in the standard library are for!

We can write a bit of generic code which exploits this; creating a `TStream` type
which implements both `Read` and `Write`.

```rust
fn reflect_stream<TStream>(stream: &mut TStream) -> std::io::Result<()>
where
    TStream: Read + Write,
```

Our code which calls `reflect_stream` keeps working because `TcpStream` implements
these traits.

An interesting standard library trait which also implements them is `Cursor`. It's
an in-memory buffer that I'm always reaching for to test streams. We can write a
simple test to assert that sending 'hello world' to the stream causes it to be written
back to the stream, which would leave two instances of it in the underlying buffer.

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::io::Cursor;

    #[test]
    fn stream_is_reflected() {
        let mut stream = Cursor::new(b"hello world".to_vec());
        reflect_stream(&mut stream).unwrap();
        assert_eq!(stream.into_inner(), b"hello worldhello world");
    }
}
```

One thing to be wary of when writing generic code like this is that the bounds can become
highly repetitive. If we're working with a type that has many `impl` blocks, we'll
be copy-pasting `Read + Write` everywhere! This is probably fine, but when many traits
are being used in the bound in complex ways, it can be noisy and confusing. Enter blanket
traits!

What we really want is a single trait – let's call it `RW` – which is both `Read` and `Write`.
We can express this with trait bounds on the definition...

```rust
trait RW: Read + Write {}

fn reflect_stream<TStream>(stream: &mut TStream) -> std::io::Result<()>
where
    TStream: RW,
```

...but while this compiles in isolation, it no longer works for any types 
(including the `TcpStream` we're already using). This is because what
we're telling the compiler here is that `RW` can only be implemented for types that 
also implement `Read` and `Write`. We haven't yet told it which types actually do 
implement `RW`, so we still need to write a manual implementation for each type 
we care about, like so:

```rust
impl RW for TcpStream {}
```

This also isn't great because we need to write a lot of boilerplate if we have many
types. We also couldn't implement this in a separate crate for other types due to the
orphan rule.

I want to avoid this. My initial attempt was to copy how I wrote the 
bounds into an `impl`... 

```rust
impl RW for Read + Write {}
```

...but this failed because you can only implement traits on types, not other traits 
(or combination of traits as shown here).

```
error[E0782]: expected a type, found a trait
  --> src/main.rs:14:13
   |
14 | impl RW for Read + Write {}
   |             ^^^^^^^^^^^^
   |
```

Instead, the magical solution is to write this very generic code...

```rust
impl<T: Read + Write> RW for T {}
```

...which is just saying that `RW` is automatically implemented for all types which
themselves implement both `Read` and `Write`.

As a more complex example, the async equivalent - using `tokio` - looks like this:

```rust
pub trait AsyncRW: AsyncRead + AsyncWrite + Unpin {}
impl<T: AsyncRead + AsyncWrite + Unpin> AsyncRW for T {}
```

Try and think of examples from your code where you could express the type required as 
a combination of traits. Blanket implementations could be a good fit there!

## My Takeaways
I like this pattern because it helps to deduplicate trait bounds – I don't need to repeat 
`AsyncRead + AsyncWrite + Unpin` everywhere!. It also makes the bound more obvious by giving it
a name; the `Unpin` bound is a good example because it's not immediately clear that would be
needed.

I've also learned to not be afraid to create my own traits. Coding against the trait makes the
code much more flexible, and using the minimum requirements for a blanket implementation makes
your code cooperative with code defined in other crates, where they perhaps can't implement your
trait. Don't be afraid also to write traits and implement them for external types, writing a 
small amount of bridging code to unify the interfaces.

Using traits in this way is beneficial to performance over trait objects because the
indirection moves from runtime (using `Box`es requires a heap allocation and following
a pointer) to compile-time (the code isn't generic at runtime).
Trait objects are often useful, but I was substantially overusing them coming from more OOP
languages, and now I can see many ways of avoiding them where they aren't necessary.
