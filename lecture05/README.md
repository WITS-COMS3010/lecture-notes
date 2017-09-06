# Thread coordination

- Sometimes multiple threads need to cooperate with each other to
  accomplish a job; in this case, we say that we need to coordinate
  the activity of the threads.
  
- The easiest way to coordiate threads is through accessing the memory
  all the treads share -- static memory and heap.

- The exact mechanism to achieve thread coordination is not trivial.

- In the lecture, we looked at the problems of thread coordination by
  writing various versions of the program in which two threads -- a
  producer thread and a consumer thread -- need to coordinate
  according to the following protocol:
  - producer is putting lines of text into a shared buffer;
  - consumer takes lines of text out of the shared buffer;
  - each line produced by the producer has to be consumed exactly once
    by the consumer.

The code is
[here](https://github.com/WITS-COMS2001/lecture-code/tree/master/lecture07). Take
a look at it, compile it, and see what happens when you run it.
- In `procon1.c`, there is no coordination between the producer and the
  consumer.
- In `procon2.c`, we try to achieve some coordination by using calls
  to `shed_yield()`.
- In `procon_flag.c`, we try achieve coordination by using a flag as
  part of the shared data structure.


For `procon_flag.c` to work correctly, two assumptions have to hold:

- we have a fair scheduler that, even on a single-core machine, will
  execute the producer and the consumer threads in turn;
- the compiler should not re-order instructions in an attempt to
  procude more efficient code; namely, it is important that the
  following sequence of instructions is executed in exactly the given
  order:
```c
so->line = line; // put the line into the shared buffer
markfull( so ); // mark the buffer as full; wait for it to become
  empty
```

Moreover, this solution works only for a single producer and single
consumer executing an alternating protocol: the consumer and producer
take turns to access the shared object; i.e., the pattern of access is
P-C-P-C-P-C ...

We want a solution that is both more general and bullet-proof.

For that purpose we use mutexes to lock access to the flag and
conditional variables to signal that the value of the flag has been
changed.  These two refinements are introduced in the files
`proNcon.c` and `proNcon2CV.c`.
