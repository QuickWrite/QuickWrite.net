+++
date = '2026-03-11T00:00:00+02:00'
draft = false
title = 'Tau'
license = "MIT"
version = "v1.0.0"
subtitle = "An esoteric Turing Machine programming language."

tags = ["Turing Machine", "Programming Language", "C"]
+++

The Tau (τ) programming language is an esoteric Turing machine programming language.
It is mainly designed to be a toy language for educational purposes to explain how turing machines work and what they are doing.

It does that by enabling the programmer to build their own turing machine and running them whilst observing what it does.

[The repository of the project can be found here.](https://github.com/QuickWrite/tau)

## Structure
The language works by being separated into two different sections that are separated with a `-`.
The first part is the definition of everything that is **not** the state-machine and the second part is the state machine itself.

### The definitions
The first part is represented as key-value pairs. These key-value pairs are defined using the `key = value` syntax.
As an example on how to design the first part with **all** possible key-value pairs, would be this:
```py
# key   = value  (Everything that starts with a # is a comment)
symbols = 0,1
blank = 0
start = A
end = HALT
tape = 1,1,0,1
```

### The state machine
After that the state machine is being defined. The state machine consists of states with different rules:
```py
# A is the name of the state
A {
    # Each of these is a rule
    # If there is a 0, then it should put 1, go right, go to state B.
    0 = 1, RIGHT, B

    # If there is a 1, then it should put 1, go left, go to state C.
    1 = 1, LEFT, C
}

B {
    0 = 1, LEFT, A
    1 = 1, RIGHT, B
}

C {
    0 = 1, LEFT, B
    1 = 1, RIGHT, HALT
}
```

Combined this would be a `.tau` file like this:
```py
symbols = 0,1
blank = 0
start = A
end = HALT
tape = 1,1,0,1

---

A {
    0 = 1, RIGHT, B
    1 = 1, LEFT, C
}

B {
    0 = 1, LEFT, A
    1 = 1, RIGHT, B
}

C {
    0 = 1, LEFT, B
    1 = 1, RIGHT, HALT
}
```

Then it can be executed using `./tau my-file.tau`.

## Documentation
There is actually a PDF file that goes into more detail on how this works. 
This file can be downloaded in the [releases of the repository](https://github.com/QuickWrite/tau/releases/).
