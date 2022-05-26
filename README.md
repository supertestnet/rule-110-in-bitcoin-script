# Rule 110 in Bitcoin Script

The following transaction spent coins from an address where the act of checking that the spend was valid required all bitcoin nodes to execute two generations of a [Rule 110 Cellular Automata](https://en.wikipedia.org/wiki/Rule_110) after being given the first generation as input: https://blockstream.info/testnet/tx/3dbdc2142f5e006a0bcd186abbb85a5f6612c8c25666ee039776ce3e970df1c7?expand

Here is the raw transaction hex: `020000000001015ebd88469de17bce4e9da9d330562f4a6792a2a13caa8a5018bb3f0038cb68e40000000000feffffff01bc02000000000000160014ff9da567e62f30ea8654fa1d5fbd47bef8e3be13050000000101e1006f7c639a916b51676b686d74518763007c6874528763007b7b686f7c639a916b51676b686d74518763007c6874528763007b7b686f7c639a916b51676b686d74518763007c6874528763007b7b686f7c639a916b51676b686d74518763007c6874528763007b7b686d756c6c6c6c006f7c639a916b51676b686d74518763007c6874528763007b7b686f7c639a916b51676b686d74518763007c6874528763007b7b686f7c639a916b51676b686d74518763007c6874528763007b7b686f7c639a916b51676b686d74518763007c6874528763007b7b686d756c6c6c6c6d6d5100000000`

# What is Rule 110?

Rule 110 is a name for a group of black and white patterns. A common one appears in this picture:

![](https://supertestnet.github.io/rule-110-in-bitcoin-script/image1.gif)

It is an object of study by computer scientists because if you modify the top line of the image, the pattern it produces changes in predictable ways, and you can use this predictable behavior to write programs that allow people to supply any sequence of black squares and white squares as input and get an output on some lower line of the image. If users supply real information to a Rule 110 program, the Rule 110 program can manipulate that information however the programmer wants it to, and you can always tell the users that they will find the result of the computation on some pre-agreed line lower down in the image. There is more information in the wiki: https://en.wikipedia.org/wiki/Rule_110

What I think is cool is that the rules for creating a Rule 110 machine can be written in Bitcoin Script and executed by bitcoin nodes.

# Does this mean bitcoin's programming language, Bitcoin Script, is turing complete?

I think so, but I am not an expert on these things and I am probably missing something. Last year a guy did a similar thing on the Bitcoin SV network and there was [an interesting thread on Hacker News](https://news.ycombinator.com/item?id=28587609) about it. In that thread, someone smarter than me thinks it doesn't count as turing complete because you have to prepare these addresses with lots of parameters that are usually not known in *normal* turing machines. For example, I knew that my program would process 4 bytes of data so I manually instructed it to halt after it processing those four bytes. A *normal* turing machine could process varying numbers of bytes, even billions of bytes. There would be some code inside the program that checks if there is anymore input and tells it to halt when there is none. But my implementation lacks that feature so maybe it's still not a real turing machine.

On the other hand, I think I could probably implement an input length checker like that very easily, and I don't think such a checker is really necessary for a machine to count as turing complete. All real-world turing machines have limited resources, such as memory, and therefore they cannot process an unlimited number of bytes. If they try to process a program that consumes more resources than they have available, their checker will never fire and the turing machine will just break after it runs out of memory or a source of energy or after its mechanical parts wear out. Bitcoin's security measures help prevent breakage by *severely* limiting the number of operations you can do, as well as how much data you can process, but since *all* real world turing machines have limits too (just much larger ones) I think it's fair to call my Rule 110 simulator a universal turing machine, even without something that checks the length of the user's input and automatically tells it to halt when all the data is processed.

# What about loops? I thought you needed loops to be turing complete, and bitcoin does not have loops

I don't think you need them. A turing machine needs to move between different states of operation where it does different things depending on the user's input, but it's okay if those states are laid out sequentially. Then the machine just progresses from e.g. state 0 to state 1 to state 2 and those states all tell it what to do depending on what the user's input is. A machine that operates in this way never needs to "loop back" to a prior state. So you don't actually need loops to simulate a turing machine -- or at least I don't think you do, but maybe I'm missing something.

# If Rule 110 is turing complete, is it useful to simulate it in Bitcoin Script?

I'm really not sure but quite possibly not. Bitcoin's limitations on script length include a rule that no more than 200 operations can be performed in a given script (with an exception for operations that only deposit a number onto the stack). A Rule 110 program that actually did anything signficant would probably need to perform hundreds of thousands of operations on the user's input, and the user's input would also need to be absolutely massive. So this is really just a toy simulator, not something you can realistically do meaningful processing with. But I don't need it to be useful. I had a lot of fun making it and to me the fun is worth the effort.

On the other hand, if I'm right that Bitcoin Script is turing complete, there are some very useful things you can do without simulating Rule 110, which is incredible inefficient as a computing device anyway. The very existence of this proof may prompt some people to seek out some of the other unexpected things Bitcoin Script can do. It's certainly got me on the watch for fantastic discoveries. The cellular automata I simulated creates an environment on the boundary between chaos and stability. To me that sounds like a pretty neat place to explore.

# How does it work?

The heart of the program is this function, which takes any sequence of 3 binary digits as input and computes an output that corresponds to what it should be on a Rule 110 table, which it then outputs to the altstack:

```
OP_SWAP
OP_IF
    OP_BOOLAND
    OP_NOT
    OP_TOALTSTACK
    OP_1
OP_ELSE
    OP_TOALTSTACK
OP_ENDIF
```

I originally implemented a Rule 110 output function that was much larger, it had 55 operations instead of 9. It caused problems because I couldn't run the function very many times without consuming the 200 operations I am allowed to use according to bitcoin's consensus rules. But then Dusty Daemon took a look at my code and suggested using boolean logic to compress the function. I googled to see if anyone had come up with a compressed way of computing Rule 110 outputs using boolean logic and I found this research paper: https://peerj.com/preprints/2553.pdf The author proves that Rule 110's output can be computed using this pseudocode:

```
If c == true:
    print(p NAND q)
Else:
    print(q)
End if
```

Which is what I implemented in Bitcoin Script above. (The extra <1> I outputed in my version is necessary so that the program always leaves 1 element on the stack, which is required by other parts of my overall program.) I am really glad that someone figured out what is probably the most efficient possible way to compute Rule 110 outputs -- standing on the shoulders of giants is the only way my program works at all.

# A few prefixes and suffixes

To compute a full line of Rule 110 the program can't just run the Rule 110 output function on 3 digits. It needs to take a string of varying length as input and run many times to compute a full line. To do this my program adds to the main function a few prefixes and suffixes. It also needs room to run a bunch of times – as many times as the inputs you provide. E.g. if the programmer wants users to provide a sequence of 10 0s and 1s as input, he or she will need to copy/paste the Rule 110 output function 10 times. He or she will also need to apply the following prefix before each run of the function: `OP_3DUP` (My rule 110 function consumes the top 2 digits of input but the bottom 2 need to be used again the next time it runs, so I duplicate all 3 and then drop 2 inputs later after the program runs.)

The program also needs the programmer to apply the following suffix after each run of the function:

```
OP_2DROP
OP_DEPTH
OP_1
OP_EQUAL
OP_IF
    OP_0
    OP_SWAP
OP_ENDIF
OP_DEPTH
OP_2
OP_EQUAL
OP_IF
    OP_0
    OP_ROT
    OP_ROT
OP_ENDIF
```

This sequence of commands cleans up the stack and loads the next set of 3 inputs so that they can be processed by the Rule 110 output function. If we are nearing the end of the stack it also adds up to two 0s to the bottom of the stack so that the OP_3DUP function won't fail when the stack runs out of elements. This is fine because Rule 110 assumes that if you are computing the output of only 2 digits, there is an imagined third one that is always a 0.

When all of these prefixes and suffixes are applied I get this result which I repeat for however many inputs there are to the program:

```
OP_3DUP
OP_SWAP
OP_IF
    OP_BOOLAND
    OP_NOT
    OP_TOALTSTACK
    OP_1
OP_ELSE
    OP_TOALTSTACK
OP_ENDIF
OP_2DROP
OP_DEPTH
OP_1
OP_EQUAL
OP_IF
    OP_0
    OP_SWAP
OP_ENDIF
OP_DEPTH
OP_2
OP_EQUAL
OP_IF
    OP_0
    OP_ROT
    OP_ROT
OP_ENDIF
```

You can see that this sequence of opcodes and numbers repeats 4 times in the top half of the witness script that I actually ran on the blockchain:

```
OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF
```

# A few more prefixes and suffixes

By repeating the Rule 110 output function 4 times, Bitcoin Script can take any line of any 4 digit Rule 110 pattern as input and compute the next line as output. But it's not ready yet, it needs a few more prefixes and suffixes and there is space to compute 2 lines so let's do that.

Earlier I said that if you are computing the output of only 2 digits using Rule 110, there is an imagined third digit that is always a 0. That's not only true when you are nearing the end of the input elements, it's also true at the beginning. Another way of saying that is this: to compute the "rightmost" digit of each line of the Rule 110 pattern, assume there is a 0 after whatever digits you supplied as input. So my function needs to deposit a zero onto the top of the stack right off the bat, which is why the first instruction in my witness script is `OP_0`. That prefix needs to be applied before you compute any line of Rule 110 so we will end up putting it on the stack twice, once at the beginning of the witness script and once in the middle.

Also, when you compute the second line of Rule 110 from the first (remember, the first line is supplied as input), the outputs are all deposited to the altstack. To use them as input for the third line of Rule 110, we need to retrieve them from the altstack. But before doing that, we need to "tidy up" the stack, because the suffix to the Rule 110 output function always adds up to two 0s to the stack when we are nearing the end of the stack, and once the line is complete we don't need those 0s anymore. So let's tidy up the stack by dropping the last two 0s and then retrieve Line 2 (which is currently on the altstack) as input for Line 3.

```
OP_2DROP
OP_DROP
OP_FROMALTSTACK
OP_FROMALTSTACK
OP_FROMALTSTACK
OP_FROMALTSTACK
```

# The whole program

If you had 10 bytes of input and if you consequently ran the Rule 110 output function 10 times, you'd need to retrieve 10 items from the altstack, but here I am only retrieving 4 items because I assume there are only 4 bytes of input. But we're almost done! By retrieving the bytes from the altstack we are back in our starting position except with a new input line, so we can duplicate what we did before to get the third line of Rule 110. The whole program looks like this:

```
OP_0 OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_2DROP OP_DROP OP_FROMALTSTACK OP_FROMALTSTACK OP_FROMALTSTACK OP_FROMALTSTACK OP_0 OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_3DUP OP_SWAP OP_IF OP_BOOLAND OP_NOT OP_TOALTSTACK OP_PUSHNUM_1 OP_ELSE OP_TOALTSTACK OP_ENDIF OP_2DROP OP_DEPTH OP_PUSHNUM_1 OP_EQUAL OP_IF OP_0 OP_SWAP OP_ENDIF OP_DEPTH OP_PUSHNUM_2 OP_EQUAL OP_IF OP_0 OP_ROT OP_ROT OP_ENDIF OP_2DROP OP_DROP OP_FROMALTSTACK OP_FROMALTSTACK OP_FROMALTSTACK OP_FROMALTSTACK OP_2DROP OP_2DROP OP_PUSHNUM_1
```

You'll notice that at the end I do op_2drop twice and I push a 1 to the stacck. I drop everything because Bitcoin nodes don't like you to have anything on the stack at the end of your function except a single item that is something other than OP_FALSE or OP_0. So I drop whatever I retrieved from the altstack and push a number 1 so that the program always succeeds as long as you provide 4 binary digits of initial input. Once it succeeds you can spend any money in the address to wherever you want. But in the process you got Bitcoin nodes to execute 2 generations of a cellular automata and you simulated a universal turing machine, so I'd say that's a job well done.
