---
layout: default
---

## Quick Start

You can explore the basics of PyMTL in a Python (>=3.6) REPL environment.

### Installing PyMTL3

```
pip install pymtl3
```

### Bits Arithmetics

Let’s start with a simple addition of two 4-bit numbers. The following code
snippet imports the basic PyMTL3 functionalities and creates two 4-bit objects
`a` and `b`:

```
>>> from pymtl3 import *
>>> a = Bits4(4)
Bits4(0x4)
>>> b = Bits4(3)
Bits4(0x3)
```

The Bits objects support common arithmetics and comparisons:

```
>>> a + b
Bits4(0x7)
>>> a - b
Bits4(0x1)
>>> a * b
Bits4(0xC)
>>> a & b
Bits4(0x0)
>>> a | b
Bits4(0x7)
>>> a > b
Bits1(0x1)
>>> a < b
Bits1(0x0)
```

### Full Adder Example

Next we will experiment with a full adder. An implementation of a full adder
has already been included in the PyMTL3 package and we can simply import it
to the REPL environment:

```
>>> from pymtl3.examples.ex00_quickstart import FullAdder
```

We can inspect the full adder implementation using the Python's dynamic inspection
feature. As you can see, the full adder logic is implemented inside an *update block*
`upblk` which is a concurrent process just like a combinational `always_comb` block in
Verilog:

```
>>> import inspect
>>> print(inspect.getsource(FullAdder))
class FullAdder( Component ):
  def construct( s ):
    s.a    = InPort()
    s.b    = InPort()
    s.cin  = InPort()
    s.sum  = OutPort()
    s.cout = OutPort()

    @update
    def upblk():
      s.sum  @= s.cin ^ s.a ^ s.b
      s.cout @= ( ( s.a ^ s.b ) & s.cin ) | ( s.a & s.b )
```

To simulate the full adder, we need to apply the `DefaultPassGroup` PyMTL pass.
Then we can set the value of input ports (through the `@=` signal assignment
operator) and simulate the full adder by calling `fa.sim_tick`:

```
>>> fa = FullAdder()
>>> fa.apply( DefaultPassGroup() )
>>> fa.sim_reset()
>>> fa.a @= 0
>>> fa.b @= 1
>>> fa.cin @= 0
>>> fa.sim_tick()
```

Now let's verify that the full adder produce the correct output:

```
>>> assert fa.sum == 1
>>> assert fa.cout == 0
```

### Register Incrementer Example

A register incrementer registers its input value and produces the output by
adding one to the registered value. Similar to the full adder, we can import
an example register incrementer like the following:

```
>>> from pymtl3.examples.ex00_quickstart import RegIncr
>>> print(inspect.getsource(RegIncr))
class RegIncr( Component ):
  def construct( s, nbits ):
    s.in_ = InPort( nbits )
    s.out = OutPort( nbits )

    s.reg_out = Wire( nbits )

    @update_ff
    def upblk_ff():
      if s.reset:
        s.reg_out <<= 0
      else:
        s.reg_out <<= s.in_

    @update
    def upblk_comb():
      s.out @= s.reg_out + 1
```

In addition to using `upblk_comb` update block to implement the increment logic,
`RegIncr` also uses `upblk_ff` flip-flop update block to register its input. Note
that PyMTL assumes each component has implicit `clk` and `reset` pins which can
be used to model synchronous posedge-triggered reset flip-flop behaviors.

The imported register incrementer implementation can be parametrized by the
bitwidth of its data path. To simulate an 8-bit register incrementer:

```
>>> regincr = RegIncr( 8 )
>>> regincr.apply( DefaultPassGroup() )
>>> regincr.sim_reset()
>>> regincr.in_ @= 42
>>> regincr.sim_tick()
```

Now let's verify that the output value is indeed incremented:

```
>>> assert regincr.out == 43
```

## About

PyMTL is developed at [Batten Research Group](https://github.com/cornell-brg), Cornell University.
