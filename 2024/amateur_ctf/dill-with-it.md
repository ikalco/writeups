

# AmateursCTF 2024 Writeup
## rev/dill-with-it (406 points)

>flocto\
>Crisp green Larry lies Bathes, brining in vinegar Dill pickle delight

### TL;DR
This challenge gives you a [python pickle](#python-pickles-and-rces) which is a binary serialization of a python object.
It's based on a virtual machine that runs custom instructions to reconstruct the object.

The virtual machine allows you to execute python, which the challenge uses to [obfuscate python code](#pain).
To deobfuscate it you must find the `pickletools` python module and use it to disassemble the pickle.
Then you can rebuild the script and do the second half of the challenge.

Once you [deobfuscate the script](#actually-reverse-engineering) you'll see that it contains a list of numbers, this is the obfuscated flag.
To deobfuscate the flag you must realize that the script uses a seeded random number generator to do
operations on the flag. After that you can simply work backwards through the operations in the script until
you get the flag. Then put all the reversed steps in a new script and you get a [solution](#solution).

### first look
```python
# Python 3.10.12
from pickle import loads
larry = b"\x80\x04ctypes\nFunctionType\n(ctypes\nCodeType\n(I1\nI0\nI0\nI4\nI8\nI67\nCbt\x00\xa0\x01|\x00d\x01\xa1\x02}\x01t\x02|\x01\x83\x01d\x00d\x00d\x02\x85\x03\x19\x00d\x00d\x03\x85\x02\x19\x00}\x00d\x04}\x02t\x03d\x05t\x04|\x00\x83\x01d\x06\x83\x03D\x00]\x11}\x03|\x02t\x05t\x00|\x00|\x03|\x03d\x06\x17\x00\x85\x02\x19\x00d\x07\x83\x02\x83\x017\x00}\x02q\x1d|\x02S\x00(NVbig\nI-1\nI-3\nV\nI0\nI8\nI2\nt(Vint\nVfrom_bytes\nVbin\nVrange\nVlen\nVchr\nt(\x8c\x04\xf0\x9f\x94\xa5\x8c\x04\xf0\x9f\xa4\xab\x8c\x04\xf0\x9f\xa7\x8f\x8c\x04\xf0\x9f\x8e\xb5tVdill-with-it\n\x8c\x04\xf0\x9f\x93\xaeI0\nC\x0c\x00\x01\x0c\x01\x1a\x01\x04\x01\x14\x01 \x01))t\x81cbuiltins\nglobals\n)R\x8c\x04\xf0\x9f\x93\xaet\x81\x940g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x05\x01.\xce\x966\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x0b\x01\xa6&\xf6\xc6v\xa6tN.\xce\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x06\x01.v\x96N\x0e\x85R\x93VWhat's the flag? \n\x85R0g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x06\x01.\xae\x0ev\x96\x85R\x93V> \n\x85R\x85R\x85R\x940g0\nC\x07\x01\xb6\xf6&v\x86N\x85Rg0\nC\x05\x01&\xa6\xa6\xce\x85R\x93Vfive nights as freddy\n\x85R0g0\nC\x07\x01\xb6\xf6&v\x86N\x85Rg0\nC\x08\x01\xa66ff\xae\x16\xce\x85R\x93g1\n\x85R0g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x05\x01.\xce\x966\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x04\x01\x0e\x86\xb6\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x0c\x01\xfa\xfaN\xf6\x1e\xfa\xfat.v\x96\x85R\x93g0\nC\x07\x01\xb6\xf6&v\x86N\x85Rg0\nC\n\x01\xce\xa6.\x9eF&v\x86N\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x04\x01v\xa66\x85R\x93g1\n\x85R\x85Rg1\n\x87R\x85R\x940g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x04\x01\x9ev\x86\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x04\x01\x0e\x86\xb6\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x0c\x01\xfa\xfaN\xf6\x1e\xfa\xfat.v\x96\x85R\x93(I138\nI13\nI157\nI66\nI68\nI12\nI223\nI147\nI198\nI223\nI92\nI172\nI59\nI56\nI27\nI117\nI173\nI21\nI190\nI210\nI44\nI194\nI23\nI169\nI57\nI136\nI5\nI120\nI106\nI255\nI192\nI98\nI64\nI124\nI59\nI18\nI124\nI97\nI62\nI168\nI181\nI61\nI164\nI22\nI187\nI251\nI110\nI214\nI250\nI218\nI213\nI71\nI206\nI159\nI212\nI169\nI208\nI21\nI236\nlg2\n\x87R\x85R\x940g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x0b\x01\xfa\xfaN\xf6\xfa\xfat.v\x96\x85R\x93g3\ng0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x0b\x01\xfa\xfa\xa6v\xfa\xfat.v\x96\x85R\x93g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x04\x01v\xa66\x85R\x93g2\n\x85RI59\n\x86R\x86R\x940g0\nC\t\x01\xcev\x96.6\x96\xaeF\x85Rg0\nC\x11\x01\xfa\xfa\xb6\xa6.\x96.\xa6\xe6\xfa\xfat.\xce\x966\x85R\x93(VLooks like you got it!\nVNah, try again.\nlg4\n\x86R."
print(loads(larry))
```

The first thing mentioned is [**Python 3.10.12**](https://www.python.org/downloads/release/python-31012/) which is weird since it's kinda old.\
Just in case we need it later we'll download and compile the tarball real quick

Then it imports the function `loads` from the `pickle` module? \
No idea what that is but a *quick google search* yields [this doc](https://docs.python.org/3.10/library/pickle.html)

> The **pickle** module implements binary protocols for serializing and de-serializing a Python object structure.

>**`pickle.loads(...)`**\
>Return the reconstituted object hierarchy of the pickled representation _data_ of an object.\
>_data_ must be a [bytes-like object](https://docs.python.org/3.10/glossary.html#term-bytes-like-object).

You'll also notice that the serialization protocol version is dependent on the python version (remember **Python 3.10.12**) and in a big red box it says that
using the module can lead to **RCE**

Ok.. so the code will deserialize the binary string into an object and print it. Right?

![WEBP of running main.py](media/dill-with-it-1.webp)

hmm..
It seems to be taking user input but an object can't do that so it's probably using that **RCE** from earlier to run `input()` or something like that

### python pickles and RCE's
Looks like its time for google again,
the first link is this an article[^1] which explains the **RCE** vulnerability of `pickle`

>.\.\.\. contains opcodes that are then one-by-one executed as soon as we load the pickle back in.

Hmm... so `pickle` doesn't actually serialize the object but instead
turns it into instructions on how to recreate the object

But how do we reverse engineer the `pickle` instructions?\
Thankfully the above article shows us that we can use `pickletools`
>If you are curious how the instructions in this pickle look like, you can use `pickletools` to create a disassembly: `pickletools.dis(pickled)`

So lets do that

![WEBP of running pickletools.dis(larry)](media/dill-with-it-2.webp)

*Oh great its 1319 lines of nonsense*

We're definitely going to need to read the docs for [`pickletools`](https://docs.python.org/3.10/library/pickletools.html) to understand this

>This module contains various constants relating to the intimate details of the [`pickle`](https://docs.python.org/3.10/library/pickle.html#module-pickle "pickle: Convert Python objects to streams of bytes and back.") module,
***some lengthy comments about the implementation***, and a few useful functions for analyzing pickled data.

The page looks pretty empty but the description mentions lengthy comments, so it must be in the [source](https://github.com/python/cpython/blob/3.10/Lib/pickletools.py), lets look at it

The very first thing in there is this comment
>Extensive comments about the pickle protocols and ***pickle-machine opcodes*** can be found here. 

and a little <kbd>Ctrl + F</kbd> verifies that the opcodes do have docs in here, awesome

Scrolling a bit further also gives a few important pieces of information
> \# "A pickle" is a program for a virtual pickle machine (PM)

> \# The PM has two data areas, "the stack" and "the memo".

> \# Many opcodes push Python objects onto the stack; e.g., INT pushes a Python
> \# integer object on the stack

> \# The memo is simply an array of objects, or it can be implemented as a dict\
> \# mapping little integers to objects.

### the instructions
Most instructions are type names followed by values, which I'm assuming just push that value to the stack like `INT`\
`INT` `UNICODE` `EMPTY_TUPLE` `TUPLE` `SHORT_BINBYTES` `SHORT_BINUNICODE`

There are a few others which aren't self explanatory, including `TUPLE` and its friends\
`MARK` `TUPLE` `REDUCE` `NEWOBJ` `GLOBAL` `STACK_GLOBAL` `GET` `MEMOIZE`

So lets look at their docs one by one\
\
`MARK` basically just acts as am marker for the start of something, Ex. tuple or list
>markobject is a unique object, used by other opcodes to identify a
      region of the stack containing a variable number of objects for them
      to work on.  See markobject.doc for more detail.

`TUPLE` creates a tuple with the values until the last `MARK` and pushes it to the stack\
`TUPLE#` are similar except they only take # values  off the stack instead of until `MARK`\
(Ex. `TUPLE1`, `TUPLE2`,  `TUPLE3`)
> All the stack entries following the topmost markobject are placed into
      a single Python tuple, which single tuple object replaces all of the
      stack from the topmost markobject onward.  For example,
\
\
> Stack before: ... markobject 1 2 3 'abc'\
> Stack after:  ... (1, 2, 3, 'abc')

`REDUCE` calls a function with a tuple of args from the stack and pushes the result onto the stack
>Push an object built from a callable and an argument tuple.\
The opcode is named to remind of the \_\_reduce\_\_() method.
\
\
Stack before: ... callable pytuple\
Stack after:  ... callable(*pytuple)

`NEWOBJ` instantiates a class based on the stack and pushes the object onto the stack, same format as reduce
>The stack before should be thought of as containing a class
      object followed by an argument tuple (the tuple being the stack
      top).  Call these cls and args.  They are popped off the stack,
      and the value returned by cls.\_\_new\_\_(cls, *args) is pushed back
      onto the stack.

`GLOBAL` pushes a class/function from a module based on a string to the stack 
> Two newline-terminated strings follow the GLOBAL opcode.  The first is
      taken as a module name, and the second as a class name.  The class
      object module.class is pushed on the stack.  More accurately, the
      object returned by self.find_class(module, class) is pushed on the
      stack, so unpickling subclasses can override this form of lookup.

`STACK_GLOBAL`[^2] this one does the same thing as `GLOBAL` except it takes the strings from the stack
>Push a global object (module.attr) on the stack.

>`STACK_GLOBAL`: take the two topmost stack items `module_name` and `qualname`, and push the result of looking up the dotted `qualname` in the module named `module_name`.

`GET`
>Read an object from the memo and push it on the stack.

`MEMOIZE`
>Store the stack top into the memo.  The stack is not popped.

### pain
Now we can actually start looking at the disassembly of the pickle,\
also the disassembly is explained in the comments on the right of it

>***btw you don't have to read all of this section, it gets annoyingly redundant after a few blocks of disassembly***

```
  2: c    GLOBAL     'types FunctionType'       # push types.FunctionType
 22: (    MARK
 23: c        GLOBAL     'types CodeType'       # push types.CodeType
 39: (        MARK
 40: I            INT        1                  # pushing a bunch of values and tuples
...
...
...
292: t            TUPLE      (MARK at 39)       # make it all into an args tuple
293: \x81     NEWOBJ                            # push CodeType.__new__(CodeType, *args)
294: c        GLOBAL     'builtins globals'     # push builting.globals
312: )        EMPTY_TUPLE                       # empty args tuple
313: R        REDUCE                            # call globals() and put it on stack
314: \x8c     SHORT_BINUNICODE '📮'
320: t        TUPLE      (MARK at 22)           # create another args tuple of 
                                                #     (CodeType, globals(), and '📮')
321: \x81 NEWOBJ                                # push FunctionType.__new__(FunctionType, *args)
322: \x94 MEMOIZE    (as 0)                     # stores new FunctionType in memo object at 0
```
The first thing it's doing is creating a function and storing it at 0 in the memo object\
Manually building the tuples back into a function yields this as 'decompiled' python code
```python
from types import CodeType
from types import FunctionType

args = (1, 0, 0, 4, 8, 67,b't\x00\xa0\x01|\x00d\x01\xa1\x02}\x01t\x02|\x01\x83\x01d\x00d\x00d\x02\x85\x03\x19\x00d\x00d\x03\x85\x02\x19\x00}\x00d\x04}\x02t\x03d\x05t\x04|\x00\x83\x01d\x06\x83\x03D\x00]\x11}\x03|\x02t\x05t\x00|\x00|\x03|\x03d\x06\x17\x00\x85\x02\x19\x00d\x07\x83\x02\x83\x017\x00}\x02q\x1d|\x02S\x00',(None, 'big', -1, -3, '', 0, 8, 2),('int', 'from_bytes', 'bin', 'range', 'len', 'chr'),('🔥', '🤫', '🧏', '🎵'),'dill-with-it', '📮', 0,b'\x00\x01\x0c\x01\x1a\x01\x04\x01\x14\x01 \x01',(), ())
code = CodeType.__new__(CodeType, *args)

args = (code, globals(), '📮')
func = FunctionType.__new__(FunctionType, *args)
```
The specifics of how the function works doesn't really matter but you'll realize that it takes a binary string and returns a normal string

The rest of the pickle uses the above function to decode binary strings into normal\
strings, which are then passed to `STACK_GLOBAL` to call actually useful functions

\
Lets continue and look at an example of that in action
```
  324: g    GET        0                                     # push our func
  327: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'
  338: \x85 TUPLE1                                           # make args tuple with bin str
  339: R    REDUCE                                           # call our func with bin str
                                                             # pushes 'builtins'
  340: g    GET        0
  343: C    SHORT_BINBYTES b'\x01.\xce\x966'
  350: \x85 TUPLE1
  351: R    REDUCE                                           # same thing but 'list'
  352: \x93 STACK_GLOBAL                                     # push builtins.list func
  353: g    GET        0
  356: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'
  367: \x85 TUPLE1
  368: R    REDUCE                                           # push 'builtins'
  369: g    GET        0
  372: C    SHORT_BINBYTES b'\x01\xa6&\xf6\xc6v\xa6tN.\xce'
  385: \x85 TUPLE1
  386: R    REDUCE                                           # push 'str.encode'
  387: \x93 STACK_GLOBAL                                     # push builtins.str.encode func
  388: g    GET        0
  391: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'
  402: \x85 TUPLE1
  403: R    REDUCE                                           # push 'builtins'
  404: g    GET        0
  407: C    SHORT_BINBYTES b'\x01.v\x96N\x0e'
  415: \x85 TUPLE1
  416: R    REDUCE                                           # push 'print'
  417: \x93 STACK_GLOBAL                                     # push builtins.print func
  418: V    UNICODE    "What's the flag? "
  437: \x85 TUPLE1                                           # create args out of str above
  438: R    REDUCE                                           # print("What's the flag? ")
  439: 0    POP
  440: g    GET        0
  443: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'
  454: \x85 TUPLE1
  455: R    REDUCE                                           # push 'builtins'
  456: g    GET        0
  459: C    SHORT_BINBYTES b'\x01.\xae\x0ev\x96'
  467: \x85 TUPLE1
  468: R    REDUCE                                           # push 'input'
  469: \x93 STACK_GLOBAL                                     # push builtins.input
  470: V    UNICODE    '> '
  474: \x85 TUPLE1                                           # create args out of str above
  475: R    REDUCE                                           # push result of input('> ')
  476: \x85 TUPLE1
  477: R    REDUCE                                           # call str.encode() with input
  478: \x85 TUPLE1
  479: R    REDUCE                                           # call list() on previous
  480: \x94 MEMOIZE    (as 1)                                # store result at 1 in memo
```
As you can see this is a very long and annoying way of doing things in python but\
it reduces down to this. Just taking user input and converting it into a list of bytes\
as integers
```python
print("What's the flag? ")
memo_1 = list(input('> ').encode())
```

\
This time I'm just going to leave the important parts
```
  485: C    SHORT_BINBYTES b'\x01\xb6\xf6&v\x86N'       # decodes to 'random'
  499: C    SHORT_BINBYTES b'\x01&\xa6\xa6\xce'         # decodes to 'seed'
  508: \x93 STACK_GLOBAL                                # push random.seed func
  509: V    UNICODE    'five nights as freddy'
  532: \x85 TUPLE1
  533: R    REDUCE                                      # random.seed('five nights as freddy')

  538: C    SHORT_BINBYTES b'\x01\xb6\xf6&v\x86N'       # decodes to 'random'
  552: C    SHORT_BINBYTES b'\x01\xa66ff\xae\x16\xce'   # decodes to 'shuffle'
  564: \x93 STACK_GLOBAL                                # push random.shuffle func
  565: g    GET        1                                # modified user input from earlier
  568: \x85 TUPLE1
  569: R    REDUCE                                      # random.shuffle(memo_1)

  574: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'  # decodes to 'builtins'
  590: C    SHORT_BINBYTES b'\x01.\xce\x966'            # decodes to 'list'
  599: \x93 STACK_GLOBAL                                # push builtins.list func

  603: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'  # decodes to 'builtins'
  619: C    SHORT_BINBYTES b'\x01\x0e\x86\xb6'          # decodes to 'map'
  627: \x93 STACK_GLOBAL                                # push builtins.map
  
  631: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'               # decodes to 'builtins'
  647: C    SHORT_BINBYTES b'\x01\xfa\xfaN\xf6\x1e\xfa\xfat.v\x96'   # decodes to 'int.__xor__'
  663: \x93 STACK_GLOBAL                                # push builtins.int.__xor__ func
  
  667: C    SHORT_BINBYTES b'\x01\xb6\xf6&v\x86N'       # decodes to 'random'
  681: C    SHORT_BINBYTES b'\x01\xce\xa6.\x9eF&v\x86N' # decodes to 'randbytes'
  695: \x93 STACK_GLOBAL                                # push random.randbytes func

  699: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'  # decodes to 'builtins'
  715: C    SHORT_BINBYTES b'\x01v\xa66'                # decodes to 'len'
  723: \x93 STACK_GLOBAL                                # push builtins.len func

  724: g    GET        1                                # get modified user input
  727: \x85 TUPLE1
  728: R    REDUCE                                      # len(memo_1)
  729: \x85 TUPLE1
  730: R    REDUCE                                      # random.randbytes(len(memo_1))
  731: g    GET        1                                # get modified user input again
  734: \x87 TUPLE3
  735: R    REDUCE             # map(int.__xor__, random.randbytes(len(memo_1)), memo_1)
  736: \x85 TUPLE1
  737: R    REDUCE             # list(map(int.__xor__, random.randbytes(len(memo_1)), memo_1))
  738: \x94 MEMOIZE    (as 2)  # store at 2 in memo
```
This block is a bit more complicated with multiple functions being passed to each other,\
but it basically seeds the python random module, then runs xor on the input with some random bytes

An important note is that we can recreate those random bytes since we know the random seed\
Anyway, here's the simplified code
```python
random.seed('five nights as freddy')
random.shuffle(memo_1)

tmp = random.randbytes(len(memo_1))
memo_2 = list(map(int.__xor__, tmp, memo_1))
```

\
This block uses `LIST` which does the same thing as `TUPLE` just into a list\
Again same thing as before, just the important parts
```
  743: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'  # decodes to 'builtins'
  759: C    SHORT_BINBYTES b'\x01\x9ev\x86'             # decodes to 'any'
  767: \x93 STACK_GLOBAL                                # push builtins.any func

  771: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'  # decodes to 'builtins'
  787: C    SHORT_BINBYTES b'\x01\x0e\x86\xb6'          # decodes to 'map'
  795: \x93 STACK_GLOBAL                                # push builtins.map func

  799: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'  # decodes to 'builtins'
                                                        # decodes to 'int.__xor__'
  815: C    SHORT_BINBYTES b'\x01\xfa\xfaN\xf6\x1e\xfa\xfat.v\x96'
  831: \x93 STACK_GLOBAL                                # push builtins.int.__xor__
  
  832: (    MARK                                        # mark start for list
  833: I        INT        138                          # a bunch more ints are added
...
 1104: l        LIST       (MARK at 832)                # make a list out of all of them

 1105: g    GET        2                                # get the xor'd input
 1108: \x87 TUPLE3
 1109: R    REDUCE                                      # map(int.__xor__, new_list, memo_2)
 1110: \x85 TUPLE1
 1111: R    REDUCE                                      # any(map(int.__xor__, new_list, memo_2))
 1112: \x94 MEMOIZE    (as 3)                           # store the result at 3 in memo
```
This block is hard to understand so I'll just write the code down and understand it later
```python
new_list = [138, 13, 157, 66, 68, 12, 223, 147, 198, 223, 92, 172, 59, 56, 27, 117, 173, 21, 190, 210, 44, 194, 23, 169, 57, 136, 5, 120, 106, 255, 192, 98, 64, 124, 59, 18, 124, 97, 62, 168, 181, 61, 164, 22, 187, 251, 110, 214, 250, 218, 213, 71, 206, 159, 212, 169, 208, 21, 236]
memo_3 = any(map(int.__xor__, new_list, memo_2))
```

\
Same thing again, just the important parts
>This is the last block !!!
```
 1117: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'         # decodes to 'builtins'
 1133: C    SHORT_BINBYTES b'\x01\xfa\xfaN\xf6\xfa\xfat.v\x96' # decodes to 'int.__or__'
 1148: \x93 STACK_GLOBAL                                       # push builtins.int.__or__
 
 1149: g    GET        3                                       # result from previous block
 
 1155: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'         # decodes to 'builtins'
 1171: C    SHORT_BINBYTES b'\x01\xfa\xfa\xa6v\xfa\xfat.v\x96' # decodes to 'int.__ne__'
 1186: \x93 STACK_GLOBAL                                       # push builtins.int.__ne__
 
 1190: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'         # decodes to 'builtins'
 1206: C    SHORT_BINBYTES b'\x01v\xa66'                       # decodes to 'len'
 1214: \x93 STACK_GLOBAL                                       # push builtins.len
 
 1215: g    GET        2               # get the original xor'd user input
 1218: \x85 TUPLE1
 1219: R    REDUCE                     # len(memo_2)
 1220: I    INT        59
 1224: \x86 TUPLE2
 1225: R    REDUCE                     # len(memo_2) != 59
 1226: \x86 TUPLE2
                                       # memo_3 was the not all zeros thing
 1227: R    REDUCE                     # memo_3 || len(memo_2) != 59
 1228: \x94 MEMOIZE    (as 4)          # store at 4 in memo

 1233: C    SHORT_BINBYTES b'\x01\xcev\x96.6\x96\xaeF'   # decodes to 'builtins'
                                                         # decodes to 'list.__getitem__'
 1249: C    SHORT_BINBYTES b'\x01\xfa\xfa\xb6\xa6.\x96.\xa6\xe6\xfa\xfat.\xce\x966'
 1270: \x93 STACK_GLOBAL                                 # push builtins.list.__getitem__
 
                           # push ['Looks like you got it!', 'Nah, try again.']
 1271: (    MARK
 1272: V        UNICODE    'Looks like you got it!'
 1296: V        UNICODE    'Nah, try again.'
 1313: l        LIST       (MARK at 1271)
 
 1314: g    GET        4               # get the boolean thing just above
 1317: \x86 TUPLE2
 1318: R    REDUCE                     # push list[memo_4]
 1319: .    STOP                       # return top of stack as result of pickle
```
This looks like it checks a few conditions and then prints whether you got the\
flag or not
```python
memo_4 = memo_3 || len(memo_2) != 59

results = ['Looks like you got it!', 'Nah, try again.']
return results[memo_4]
```

### actually reverse engineering
Now we can combine all the snippets into a script and continue
>btw I'll add nice names
```python
import random

# get user input
print("What's the flag? ")
user_input = list(input('> ').encode())

# initialize random number generator
random.seed('five nights as freddy')

# shuffle randomly and then xor user input with random bytes
random.shuffle(user_input)
random_bytes = random.randbytes(len(user_input))
xored_input = list(map(int.__xor__, random_bytes, user_input))

# xor the list with the xored input
new_list = [138, 13, 157, 66, 68, 12, 223, 147, 198, 223, 92, 172, 59, 56, 27, 117, 173, 21, 190, 210, 44, 194, 23, 169, 57, 136, 5, 120, 106, 255, 192, 98, 64, 124, 59, 18, 124, 97, 62, 168, 181, 61, 164, 22, 187, 251, 110, 214, 250, 218, 213, 71, 206, 159, 212, 169, 208, 21, 236]
xored_list = map(int.__xor__, new_list, xored_input)

# check wierd conditions
not_got_flag = any(xored_list) || len(xored_input) != 59

# return the result from the pickle
results = ['Looks like you got it!', 'Nah, try again.']
return results[not_got_flag]
```
hmm...\
The script looks like it takes `user_input`, does some stuff to it with `new_list` and random, and then checks whether you got the flag or not.

I'm not exactly sure what the script does but maybe we can start with the
condition where we got the flag and it returns `'Looks like you got it'`,
then just **work backwards**.

###

We know that `not_got_flag` will need to be `False` so that `results[False]`
will be `results[0]` and it will return `'Looks like you go it'`

From there we know that `not_got_flag`'s definition will need to be `False`\
`not_got_flag = any(xored_list) || len(xored_input) != 59` -> `False`

and for `not_got_flag` to be `False` we need **both `any(xored_list)`
and `len(xored_input) != 59` to be `False`**

###

Lets start with the easy part, we want `len(xored_input) != 59` to be `False`

If we make `xored_input` 59 characters long, then `len(xored_input)` will
return 59, and we know that `59 != 59` will be `False`

###

Now for the hard part, we want `any(xored_list)` to return `False`

First let's understand the `any` function, here's the [docs](https://docs.python.org/3.10/library/functions.html#any)
>Return `True` if any element of the _iterable_ is true. If the iterable is empty, return `False`.

it checks if any of the elements in `xored_list` are `True`,
which means every value in `xored_list` must be `False` for it to return `False`

But `xored_list` is a list of integers... how does python check if an integer is `False`?
>In Python, **the integer `0` is always `False`**, while every other number, _including negative numbers_, are `True`.

So, to have every value in `xored_list` be `False`,
**every integer in `xored_list` needs to be `0`**

###

To understand how to do that lets look at the definition of `xored_list`\
`xored_list = map(int.__xor__, new_list, xored_input)`

So, `xored_list` is made from xoring every value of `new_list` with
every corresponding value of `xored_input`

But how do we get xor to return `0` for every value in `new_list` and `xored_input`?

*google*[^3] `how to get 0 out of xor`
>XOR is a logical operator that works on bits. Let’s denote it by `^`.
>**If the two bits it takes as input are the same, the result is `0`**, otherwise it is `1`.

So, to have xor return `0` for every value in `new_list` and `xored_input`,
**every value in `new_list` needs to be equal to every value `xored_input`**

###

Let's start with the definition of `xored_input`, except we'll replace it with
`new_list` since they must be equal to each other\
`new_list = list(map(int.__xor__, random_bytes, user_input))`

Once again, like `xored_input`, we're xoring every value of `random_bytes`\
with every corresponding value of `user_input`.

But this time we want `user_input`, so how do we do that?\
First let's write it out simply so we can understand it more clearly\
`new_list = random_bytes ^ user_input`

hmm.. maybe we can inverse the xor and isolate `user_input`?

*google*[^4] `how to inverse xor`
>The inverse is XOR!
If you have:
>```java
>c = a^b;
>```
>You can get  `a`  or  `b`  back if you have the other value available:
>```java
>a = c^b; // or b^c (order is not important)
>b = c^a; // or a^c
>```

great!\
Using this principle we can figure out that
`new_list ^ random_bytes = user_input`
or, if we rearrange,
**`user_input = random_bytes ^ new_list`**

###

Now we need to find out what `random_bytes` is in order to xor it with
`new_list` and get `user_input`. But how do we do that? It's random...

Luckily the random number generator is seeded earlier in the script, which means
that `random_bytes` isn't actually random and we can figure out it's value.

To do so we can't just run `random.randbytes()` because `random.shuffle()`
is run earlier in the script which causes the random number generator to spit out
different values.

What we need to do is simulate the random number generator shuffling something
in the same way as was done originally, so that the correct bytes that are spit out by
`random.randbytes()`

For example
```python
random.seed('seed')
random.shuffle(list('hello'))
rand1 = random.randbytes(1)

random.seed('seed')
random.shuffle(list('a' * 5))
rand2 = random.randbytes(1)
```
In this example `rand1` will be the same as `rand2` because `random.shuffle()` was
used in the same way when both were generated.

You may say that they aren't the same since the characters aren't the same but the
characters don't matter. It's only the way the letters were shuffled that matters, and
therefore the only thing needed is for the length of the two shuffled strings to be the same

###

Now that we know `random_bytes` we can xor it with `new_list` and get `user_input`.
But it won't actually be the original user input, but instead a shuffled version of it due to the
`random.shuffle(user_input)`.

There isn't a built in way to unshuffle something in python so we'll have to find a way to do it.

*google*[^5] `how to undo random.shuffle in python`

>```python
>import random
>
>def shuffle_under_seed(ls, seed):
>   # Shuffle the list ls using the seed `seed`
>   random.seed(seed)
>   random.shuffle(ls)
>   return ls
>
>def unshuffle_list(shuffled_ls, seed):
>   n = len(shuffled_ls)
>   # Perm is [1, 2, ..., n]
>   perm = [i for i in range(1, n + 1)]
>   # Apply sigma to perm
>   shuffled_perm = shuffle_under_seed(perm, seed)
>   # Zip and unshuffle
>   zipped_ls = list(zip(shuffled_ls, shuffled_perm))
>   zipped_ls.sort(key=lambda x: x[1])
>   return [a for (a, b) in ls]
>```
This code basically just figures out where each element in a known list was moved, and
then does the opposite to the given list in order to unshuffle it.

###

Finally we have the unshuffled `user_input`, but it's not a string. It's a list of integers.
To turn it back into a string we need to interpret the integers as ASCII bytes using `bytes()`
and then we can run `.decode()` on it in order to turn it from a byte string into a normal string

### putting it all together

First let's define what we know
* **we need the `random` module**
* **the length of `user_input` should be `59`**
* **we know the numbers in `new_list`**
* **we know how to unshuffle a list**

```python
import random

def shuffle_under_seed(ls, seed):
  # Shuffle the list ls using the seed `seed`
  random.seed(seed)
  random.shuffle(ls)
  return ls

def unshuffle_list(shuffled_ls, seed):
  n = len(shuffled_ls)
  # Perm is [1, 2, ..., n]
  perm = [i for i in range(1, n + 1)]
  # Apply sigma to perm
  shuffled_perm = shuffle_under_seed(perm, seed)
  # Zip and unshuffle
  zipped_ls = list(zip(shuffled_ls, shuffled_perm))
  zipped_ls.sort(key=lambda x: x[1])
  return [a for (a, b) in zipped_ls]

flag_len = 59
new_list = [138, 13, 157, 66, 68, 12, 223, 147, 198, 223, 92, 172, 59, 56, 27, 117, 173, 21, 190, 210, 44, 194, 23, 169, 57, 136, 5, 120, 106, 255, 192, 98, 64, 124, 59, 18, 124, 97, 62, 168, 181, 61, 164, 22, 187, 251, 110, 214, 250, 218, 213, 71, 206, 159, 212, 169, 208, 21, 236]
```

next we need to get `user_input`, but to do that we need to
* **regenerate `random_bytes` using the seed**
* **`user_input = random_bytes ^ new_list`**
* **unshuffle `user_input` into the flag**

```python
random.seed('five nights as freddy')
random.shuffle(list('a' * flag_len))
random_bytes = random.randbytes(flag_len)

user_input_shuffled = list(map(int.__xor__, random_bytes, new_list))
user_input = unshuffle_list(user_input_shuffled, 'five nights as freddy')
```

finally we just need to convert `user_input` from a list of integers into a string

```python
flag = bytes(user_input).decode()
flag = ''.join(flag)
print(flag)
```

### solution

<details>
<summary>:checkered_flag:</summary>
amateursCTF{p1ckL3-is_not_the_goat_l4rrY_is_m0R3_\:goat:ed}
</details>

```python
import random

def shuffle_under_seed(ls, seed):
  # Shuffle the list ls using the seed `seed`
  random.seed(seed)
  random.shuffle(ls)
  return ls

def unshuffle_list(shuffled_ls, seed):
  n = len(shuffled_ls)
  # Perm is [1, 2, ..., n]
  perm = [i for i in range(1, n + 1)]
  # Apply sigma to perm
  shuffled_perm = shuffle_under_seed(perm, seed)
  # Zip and unshuffle
  zipped_ls = list(zip(shuffled_ls, shuffled_perm))
  zipped_ls.sort(key=lambda x: x[1])
  return [a for (a, b) in zipped_ls]

flag_len = 59
new_list = [138, 13, 157, 66, 68, 12, 223, 147, 198, 223, 92, 172, 59, 56, 27, 117, 173, 21, 190, 210, 44, 194, 23, 169, 57, 136, 5, 120, 106, 255, 192, 98, 64, 124, 59, 18, 124, 97, 62, 168, 181, 61, 164, 22, 187, 251, 110, 214, 250, 218, 213, 71, 206, 159, 212, 169, 208, 21, 236]

random.seed('five nights as freddy')
random.shuffle(list('a' * flag_len))
random_bytes = random.randbytes(flag_len)

user_input_shuffled = list(map(int.__xor__, random_bytes, new_list))
user_input = unshuffle_list(user_input_shuffled, 'five nights as freddy')

flag = bytes(user_input).decode()
flag = ''.join(flag)
print(flag)
```

### other links
>Some of these links may not work in the future btw

[^1]: [python pickle article](https://davidhamann.de/2020/04/05/exploiting-python-pickle/#how-to-dump-and-load)

[^2]: [STACK_GLOBAL explanation](https://peps.python.org/pep-3154/#summary-of-new-opcodes)

[^3]: [0 from xor](https://florian.github.io/xor-trick/#xor)

[^4]: [inverse xor](https://stackoverflow.com/questions/14279866/what-is-inverse-function-to-xor)

[^5]: [how to unshuffle](https://crypto.stackexchange.com/questions/78309/how-to-get-the-original-list-back-given-a-shuffled-list-and-seed)

