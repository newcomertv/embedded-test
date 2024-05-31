Most common errors:



## `error[E0463]: can't find crate for test`

This can happen if you invoke `cargo test` without any additional arguments. Apparently `cargo test` wants to also test the binaries... 

Adding the following to `Cargo.toml` helps as a workaround.

```diff
[[bin]]
name = "embedded-example-test"
path = "src/main.rs"
+ test = false
```

Alternatively, you can also run `cargo test --tests` to only run the integration tests.

## `rust-lld: error: undefined symbol: embedded_test_linker_file_not_added_to_rustflags`

Make sure you've added the following line to your `build.rs` linker script:

```rust
println!("cargo::rustc-link-arg-tests=-Tembedded-test.x");
```

You can also add `rustflags = [ "-C", "link-arg=-Tembedded-test.x" ]` to your `.cargo/config.toml` file. But then you need to move `embedded-test` from `[dev-dependencies]` to `[dependencies]` in your `Cargo.toml`.