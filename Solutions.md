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