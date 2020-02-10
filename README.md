This is an example of how to make a self-contained cdylib with a custom entry point.

# Crate Type

First, we need to set the crate type:

```toml
# Cargo.toml

[lib]
crate-type = ["cdylib"]
```

# No Standard

Because we don't want to link to system libraries, we can't use the Rust
standard library:

```rust
// src/lib.rs

#![no_std]
```

# Abort on Panic

Since we aren't using the standard library, we have to abort the process
during a panic. This requires changes in both `Cargo.toml` and the library.
Note that these configuration changes in `Cargo.toml` won't work inside a
workspace. They only work at the top-level of the workspace.

```toml
# Cargo.toml

[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

```rust
// src/lib.rs

#[panic_handler]
fn panic(_info: &core::panic::PanicInfo) -> ! {
    loop {}
}
```

# Custom Entry Point

Finally, we need to specify the custom entry point. This happens pretty
automatically by simply defining a `_start` symbol in the binary. However,
this often times can't be Rust (for example, a kernel).

First, we'll use the `cc` crate to build and link some assembly.

```toml
# Cargo.toml

[build-dependencies]
cc = "*"
```

Then we'll add a build script to compile our assembly code.

```rust
// build.rs

fn main() {
    cc::Build::new().file("src/start.s").compile("start");
}
```

Finally, we'll add our assembly.

```
# src/start.s

      .text
      .globl _start
      .type _start, @function
_start:
    ud2 # Crash immediately...
```
