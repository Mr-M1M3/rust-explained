# RefCell

```rust
// RefCell lets us mutate immutable data by asking ask rust to check borrowing rules at runtime.
use std::cell::RefCell;
    // if we decalre a variable without mut keyword:
    let a = String::from("nothing");

    // -> we cannot mutate the data ( by that I mean, we cannot chnage the internal state of the data that variable holds)
    // a.push_str("pushed str"); ❌

    // -> we cannot reassign the varible
    // a = String::new(); ❌

    // -> We can shadow it
    //let a = String::new(); // ✅

    // if we decalre a variable using mut keyword:
    let mut a_mut = String::from("nothing");

    // -> we can mutate the data ( by that I mean, we cannot chnage the internal state of the data that variable holds)
    //a_mut.push_str("pushed str"); // ✅

    // -> we cannot reassign the variable
    //a_mut = String::new(); // ✅

    // -> We can shadow it
    //let a = String::new(); //✅

    // What RefCell lets us is mutate the data it holds. it doesn't care whether the data it holds is marked mutable via mut keyword
    let rand_str = String::from("I am immutable.");
    // we can't mutate it
    // rand_str.push_str("i was made to be pushed"); ❌

    //but if we **move** that immutable data to a refcell
    let str_inside_refcell = RefCell::new(rand_str);
    // we can mutate the data that refcell holds
    str_inside_refcell
        .borrow_mut() // borrows  the data refcell holds mutably in runtimne
        .push_str(" or am I?");
    println!("{}", str_inside_refcell.borrow());

    // println!("{rand_str}"); ❌ // we moved rand_str, can't use it anymore.

    // we can't reassign the variable that holds refcell either
    // str_inside_refcell = RefCell::new(String::new()); ❌

    // we can't reassign rand_str either because rand_str was declared without mut keuword
    // rand_str = String::new(); ❌

    // but if we decalre a variable using mut keyword
    let mut mut_rand_str = String::from("mut str");
    // then, we passed that to refcell
    let mut_str_in_refcell = RefCell::new(mut_rand_str);

    // println!("{mut_rand_str}"); ❌ // we moved mut_rand_str

    // we can't reassign the variable taht holds refcell either
    // mut_str_in_refcell = RefCell::new(String::new()); ❌

    //  but *** WE CAN REASSIGN mut_rand_str *** because mut_rand_str was declared using mut keuword.
    // mut_rand_str = String::new(); ✅

    /*  if
        -> we pass ownership to RefCell
            -> we can call .borrow() ✅
            -> we can call .borrow_mut() ✅
        -> we pass a refrence to RefCell
            -> immutable reference
                -> we can only call .borrow() ✅
                -> we cannot call .borrow_mut() ❌
            -> mutable reference
                -> we can call .borrow() ✅
                -> we can call .borrow_mut() ✅
    */

    /*  note, tho we can but it doesn't make sense to wrap literal types (chars, integars, floats) with RefCell
        because they don't have any internal states to change.
    */
```
