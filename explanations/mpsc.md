# mpsc

```rust
use std::{sync::mpsc, thread};

fn main() {
    //You can imagine a channel in programming as being like a directional channel of water,
    //such as a stream or a river. If you put something like a rubber duck into a river,
    //it will travel downstream to the end of the waterway.
    //channel has two halves: a transmitter and a receiver.
    //The transmitter half is the upstream location where you put the rubber duck into the river,
    //and the receiver half is where the rubber duck ends up downstream.
    //One part of your code calls methods on the transmitter with the data you want to send,
    //and another part checks the receiving end for arriving messages.
    //A channel is said to be closed if either the transmitter or receiver half is dropped.

    {
        // isolated using code blocks to illustrate different cases
        let (tx, rx) = mpsc::channel::<i32>();

        let thread = thread::spawn(move || {
            // rx.recv()
            let rcvd = rx.recv().expect("channel closed"); // .reecv() blocks the thread and prevents for going to the next line
            // .recv() only returns Err variant if the channel is found closed

            println!("{rcvd}"); //never gets executed
        });

        thread.join().unwrap(); // we wait for the spawned thread to finish
        // but the spawned thread waits for us to send a msg
        // we never pass a msg
        //spawned thread waits forever and never finishes
        //that's why main thread waits forever too.
    }

    {
        // isolated using code blocks to illustrate different cases
        let (tx, rx) = mpsc::channel::<i32>();

        let thread = thread::spawn(move || {
            // rx.recv()
            let rcvd = rx.recv().expect("channel closed");
            println!("{rcvd}");
        });
        tx.send(100).expect("channel is closed"); //we passed a message
        //spawned thread gets the message and immediately goes to the next line
        // gets to the last line of the closure body and finishes executing the closure
        //because the closure is finished executing, spawned thread is considered to be done
        thread.join().unwrap(); // we wait for the spawned thread to finish
        // after the spawned thread is done
        //main thread exits as this is the last line of the main thread
    }

    {
        // isolated using code blocks to illustrate different cases
        let (tx, rx) = mpsc::channel::<i32>();

        let thread = thread::spawn(move || {
            // rx.recv()
            let rcvd = rx.recv().expect("channel closed");
            println!("{rcvd}");
        });
        tx.send(100).expect("channel is closed"); //we passed a message
        //spawned thread gets the message and immediately goes to the next line
        // gets to the last line of the closure body and finishes executing the closure
        //because the closure is finished executing, spawned thread is considered to be done
        //after the thread is done, rx gets droppped
        tx.send(100).expect("channel is closed"); // panics ‼️
        // because rx gets dropped, the channel is closed
        //and we can't send more messages
        thread.join().unwrap(); //main thread panics and exits before we get to this line
    }

    // So what do we do to send more than one message?
    {
        // isolated using code blocks to illustrate different cases
        let (tx, rx) = mpsc::channel::<i32>();

        let thread = thread::spawn(move || {
            // we loop forever and keep listening for new messages
            loop {
                let rcvd = rx.recv().expect("channel closed"); // blocks the thread and waitrs for a message.s
                //next lines of code is not executed until a message is rcvd
                println!("{rcvd}");
            } // after the loop body gets executed, goes to the top bof the loop body and waits for a message again
        });
        tx.send(100).expect("channel is closed"); //we passed a message

        tx.send(100).expect("channel is closed"); // we passed another message
        //because the spawned thread loops forever, we can pass as many msgs we want
        for i in 1..=10 {
            tx.send(i).unwrap();
        }
        thread.join().unwrap(); // we wait for the spawned thread to finish
        // but the closure we passed loops forever and never finshes
        //that's why main thread waits forever and never exits
    }

    // what to do if we no longer need the spawned thread
    // drop the sender
    {
        // isolated using code blocks to illustrate different cases
        let (tx, rx) = mpsc::channel::<i32>();

        let thread = thread::spawn(move || {
            // we loop forever and keep listening for new messages
            loop {
                let rcvd = rx.recv().expect("channel closed"); // blocks the thread and waitrs for a message.s
                //next lines of code is not executed until a message is rcvd
                println!("{rcvd}");
            } // after the loop body gets executed, goes to the top bof the loop body and waits for a message again
        });
        tx.send(100).expect("channel is closed"); //we passed a message

        tx.send(100).expect("channel is closed"); // we passed another message
        //because the spawned thread loops forever, we can pass as many msgs we want
        for i in 1..=10 {
            tx.send(i).unwrap();
        }
        drop(tx);// this closes the channel causing to .recv().expect(...) panic and exit the spawned thread (not only the loop body)
        thread.join().unwrap(); // we wait for the spawned thread to finish
        // but the closure we passed loops forever and never finshes
        //that's why main thread waits forever and never exits
    }

    //better approach
    // what to do if we no longer need the spawned thread
    // conditionally break ou of the loop
    {
        enum Command<T> {
            Process(T),
            Terminate,
        }
        // isolated using code blocks to illustrate different cases
        let (tx, rx) = mpsc::channel::<Command<i32>>();

        let thread = thread::spawn(move || {
            // we loop forever and keep listening for new messages
            loop {
                let rcvd = rx.recv().expect("channel closed"); // blocks the thread and waitrs for a message
                // after getting a message goes to the next line, we check what command we got
                match rcvd {
                    // if we are told to process
                    Command::Process(i) => {
                        println!("{i}");
                    }
                    Command::Terminate => {
                        break; // we break out of the loop if we are told to terminate
                    }
                }
            }
            //if we break out from the loop, there is nothing left to be executed and the ckosure finishes executing
        });
        tx.send(Command::Process(100)).expect("channel is closed"); //we passed a message

        tx.send(Command::Process(200)).expect("channel is closed"); // we passed another message
        //we can pass as many msgs we want
        for i in 1..=10 {
            tx.send(Command::Process(i)).unwrap();
        }
        //when we no longer need the thread,
        tx.send(Command::Terminate).unwrap(); // this will ask the spawne thread to break out of the loop
        //after bvreaking out of the loop, the closure finishes excuting and the spawned threadf is considered done
        thread.join().unwrap(); // we wait for the spawned thread to finish
        // after the spawned thread is done
        //main thread exits as this is the last line of the main thread
    }

    // There is a better approach
    //because rx is of type `std::sync::mpsc::Receiver<T>` which implements Iterator trait, we can just ityerate over it
    {
        // isolated using code blocks to illustrate different cases
        let (tx, rx) = mpsc::channel::<i32>();

        let thread = thread::spawn(move || {
            /* calling rx.next()
                  -> if a value is already on the channel yield the value
                  -> if no value is present on the channel, block the thread
                  -> if the channel is closed, instead of throwing an Err variant (like .recv() does), it yields
                     None which indicates the iteration is complete
            */
            for rcvd in rx {
                println!("{rcvd}");
            }
        });
        // now we can send as many messages as we want
        for i in 1..=10 {
            tx.send(i).unwrap();
        }
        thread.join().unwrap(); // we wait for the spawned thread to finish
        // but the spawned thread waits forever because the channle is never closed resulting in infinite itration.
        //that's why main thread waits forever too.
    }
    // what to do after we arew done sending messages?
    // juist like we did when we were using loop block,
    // conditionally break from the iteration
    // or we could just drop the sender by calling
    //drop(tx)
}
```
