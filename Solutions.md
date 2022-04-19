## Lesson 1
we start with an empty stack
we pass `CALLVALUE` 8, putting it to the top of the stack
`JUMP` consumes the top value and jumps to the `nth` position in the stack, which in our case is `08    5B    JUMPDEST`

## Lesson 2
We're going to work backwards to solve this one. Like lesson 1, we know we need to `JUMP` - this time to position 6. This means that we need the result of the call to `SUB` to be 6. 
`SUB` subtracts the first element from the stack from the second element.
* The first element will be the result of `CODESIZE` (the number of instructions in the code), which is **10**
* The second elemtn will be the result of `CALLVALUE`

This means `6 = 10 - CALLVALUE`, where `CALLVALUE` = **4**.

## Lesson 3
Our goal is clear here, we need to pass `calldata` with `CALLDATASIZE` = **4**.

If we look at the [evm.codes](https://www.evm.codes/) site, we can see that the `CALLDATASIZE` of '0xff' represents a byte size of **1**, so all we need to do is pass 4 bytes to `calldata` - that is, `'0xffffffff'.

## Lesson 4

Again, we need to pass `JUMP` the correct value, which in this case is **10** (0A). This time, this hinges on the `XOR` of whatever `CALLVALUE` we pass and the `CODESIZE`, which is **12** (0c). So our formula is as follows:`0c XOR CALLVALUE = 10`.

This requires knowledge of `xor`s, a binary operation. We can re-write our formula as `1100 xor CALLVALUE = 1010`. To flip the bits of `1100` -> `1010`, the correct answer is `0110`, which in hexidecimal is `06`!

## Lesson 5
Ooh ok, this one is a little more involved. We supply a `CALLVALUE` which gets duped (`DUP1`), those two then get multiplied (`MUL`). Then, a two byte 0100 is pushed to the top of the stack (`PUSH2`) which must equal what we got from the `MUL` (`EQ`). Lastly, 12 (0C) is pushed to the top of the stack and `JUMPI` conditionally takes us to `JUMPDEST` if the result of `EQ` == **1**.

So to crack this, we just need to make the `sqrt(CALLVALUE) == PUSH2 0100`, **where 0100 = 256** in decimal. This evaluates to **16**.

## Lesson 6
New opcode `CALLDATALOAD`! The first two steps do the following: `CALLDATALOAD` takes the input at the top of the stack 00 (from `PUSH1 00`) and returns call data from that byte onward. So in our case, that's byte 0-32 - this gets right padded btw. We need to send `JUMP` to 10 (`0A`), which means that our 32 byte calldata must evaluate to 10.

Because calldata gets right padded we can't just enter `0x0a`, so to solve this, enter `0x000000000000000000000000000000000000000000000000000000000000000a`.

## Lesson 7
Definitely the most steps we've seen so far! Skipping to the `CREATE` code - this takes the top of the stack `[value offset size]` and then pushes the address that the account was deployed to to the top of the stack. Next, `EXTCODESIZE` returns the size of the code at that address, followed by a `PUSH 01` and `EQ`. 

Bringing it all together, we need to enter calldata such that the code size is equal to **01 byte**. After some brute forcing on [evm.codes](https://www.evm.codes/playground?unit=Wei&codeType=Mnemonic&code=%27y1z10z10twwy2v32%200xssssz2t%27~uuuuzv1%20y%2F%2F%20Example%20w%5CnvwPUSHuFFtwMULs~~%01stuvwyz~_), `0x60016000526001601ff3` solved it for me.

## Lesson 8
We see a very similar `CALLDATASIZE PUSH1 00 DUP1 CALLDATACOPY CALLDATASIZE PUSH1 00 PUSH1 00 CREATE`, which just creates a new contract from the calldata and returns a deployment address. At this point we have a stack that looks like: `[address_deployed 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]`. 

The next 5 instructions have to do with the `CALL` instruction which executes the code of the given account in a new subcontext. This opcode expects the following at the top of the stack: `[gas address value argsOffset argsSize retOffset retSize]`.

After the next `PUSH1` and `DUP1`'s, our stack looks like this `[0 0 0 0 0 address_deployed 0 0 0 0 0 0 0 0 0 0]`. Then, we get to the `SWAP5` opcode which swaps stack item 1 <-> 6 and finally `CALL` is executed which returns **0** if reverted or **1** on success. 

After the `CALL` instructions we see `PUSH 00 EQ`, meaning we need `CALL` to push a **0** onto the stack. 

To solve, pass in calldata with a bytecode sequence such that the return value of the sequence causes a REVERT when run, ex: **0x60016000526001601ff3**