# Variable Declaration in Rust

### Declaring variables without mut keyword

```rust
let literal = 32;
let owned = String::from("random string");
```

- ⚠️ You cannot reassign the variable

```rust
literal = 100; // ❌
owned = String::from("reassigned"); // ❌
```

- ⚠️ You cannot mutate the data

```rust
owned.push_str("pushed string");// ❌
```

- ⚠️ You cannot take mutable reference to the data

```rust
let m_ref_literal = &mut literal; // ❌
let m_ref_owned = &mut owned; // ❌
```

You can however,

- Shadow the variable

```rust
let literal = 100;// ✅
let owned = String::from("shadowed");// ✅
```

- take immutable reference to the data

```rust
let im_ref_literal = &literal; // ✅
let im_ref_owned = &owned; // ✅
```

### Declaring variables _with_ mut keyword

```rust
let mut literal = 32;
let mut owned = String::from("random string");
```

- You can reassign the variable

```rust
literal = 100; // ✅
owned = String::from("reassigned");//✅
```

- You can mutate the data

```rust
owned.push_str("pushed string");// ✅
```

- You can take immutable reference to the data

```rust
let im_ref_literal = &literal; // ✅
let im_ref_owned = &owned; // ✅
```

- You can take mutable reference to the data

```rust
let m_ref_literal = &mut literal; // ✅
let m_ref_owned = &mut owned; // ✅
```

- You can shadow the variable

```rust
let literal = 100;// ✅
let owned = String::from("shadowed");// ✅
```

### What if you take an immutable reference and bind it to a mutable variable

You can only reassign and shadow the variable or just take another reference to the reference itself.
You cannot mutate what the reference refers to.

```rust
let referred_data = String::from("i was referred to");

let mut im_ref_in_m_var = &referred_data;

let a = String::from("my ref was reassigned");
im_ref_in_m_var = &a; // ✅
let im_ref_in_m_var = String::from("i shadowed"); // ✅

```

- ⚠️ you cannot mutate the data it refers to

```rust
im_ref_in_m_var.push_str("can't do that"); // ❌
*im_ref_in_m_var = String::new(""); // ❌
```

- Theoretically, you can take a mutable reference but, it doesn't make any sense since you cannot really mutate that by the newly taken mutable reference.

### What if you take a mutable reference and bind it to an immutable variable

You just cannot reassign the variable that holds the reference. Other than that, you should be able to do whatever you want.

```rust
let mut referred_data = String::from("i was referred to");

let im_ref_in_m_var = &mut referred_data;

let a = String::from("my ref was reassigned");
im_ref_in_m_var = &a; // ❌
```

- you can mutate the data it refers to

```rust
im_ref_in_m_var.push_str("can't do that"); // ✅
*im_ref_in_m_var = String::new(); // ✅ (basically we are reassigning the `im_ref_in_m_var` variable)
```

> A reference is just a pointer. It doesn't hold the actual data. It stores the memory location of another data.
> When we use dereference operation on a reference, we just ask the compiler to follow the location and get the data the reference refers to.
> So, it doesn't matter where we store the reference. When we try to use the reference and to do that if rust needs to follow the reference, it will enforce ownership and borrowing rules of the data it refers to. Otherwise rust will just enforce ownership and borrowing rules of the variable that holds the reference.
