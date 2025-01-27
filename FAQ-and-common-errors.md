Most common errors:



## `error[E0463]: can't find crate for test`

This can happen if you forget to add `harness = false` to your `Cargo.toml` for each test.


Furthermore, this error can happen if you invoke `cargo test` without any additional arguments. Apparently `cargo test` wants to also test the binaries... 

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

## `rust-lld: error: duplicate symbol: _SEGGER_RTT`

You probably have the `init-log` feature activated on `embedded-test` while at the same time using a crate like `defmt-rtt` or `rtt-log`. You need to either setup the logger yourselve (with a crate of your choice) or let embedded-test do it (`init-log` feature)

## probe-rs exits directly after flashing (semihosting...)

Make sure you're using at least probe-rs 0.24.0. Older versions do not support embedded-test yet.

## probe-rs complains about unexpected arguments

Make sure you're using at least probe-rs 0.24.0. Older versions do not support embedded-test yet.  
Otherwise, please open an issue including the commandline that was used to invoke probe-rs.

## probe-rs fails to link when using `embedded-test` with `defmt`

```log
  = note: rust-lld: error: undefined symbol: _defmt_release
          >>> referenced by mod.rs:71
          ...
```

If only your test builds fail but your main application builds fine, you probably have some linker scripts that are not linked for the tests but are linked for the main application.

Check your `build.rs` for `println!("cargo:rustc-link-arg-bins=-Tdefmt.x");` or some other dependency which might be missing.
Add `println!("cargo:rustc-link-arg-tests=-Tdefmt.x");` or some other dependency which might be missing. to your `build.rs`.

## probe-rs fails to link when using `embedded-test` with `defmt-rtt`

If your using `defmt-rtt` rather than `rtt-target` you will have to use `defmt-rtt` within your test code.
This can be done with `use defmt_rtt as _;` at the top of your test file.
