# bomb_lab_solution

In this repository I will take down my process of solving the bomb lab of CS:APP. Since there exists a bunch of different versions of this problem, I' ve already uploaded my version. Although the problems differ from each other, the main methods we take are totally the same.

## Prerequisites
1. GDB (debug tool)
2. objdump  (command)
3. strings (command)

## Step 0
In this part we use objdump to get the assembly code
we use 
``` 
objdump -d bomb > bomb.asm
```
and get the following file (not the full code)
``` asm
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq   

```
Then we go and defuse the first bomb. 

## Step 1
We enter gdb, set a breakpoint at the phase 1.
Then we take a look at the assembly code above, we see one register eax and an address 0x402400. 
Enter a random string and then we stop at the phase 1 position, then we try printing out the information around 0x402400. 
We get the following part
``` 
(gdb) p/x $eax
$1 = 0x603780
(gdb) x /25c 0x603780
0x603780 <input_strings>:	116 't'	101 'e'	115 's'	116 't'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x603788 <input_strings+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x603790 <input_strings+16>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x603798 <input_strings+24>:	0 '\000'
(gdb) x /25c 0x402400
0x402400:	66 'B'	111 'o'	114 'r'	100 'd'	101 'e'	114 'r'	32 ' '	114 'r'
0x402408:	101 'e'	108 'l'	97 'a'	116 't'	105 'i'	111 'o'	110 'n'	115 's'
0x402410:	32 ' '	119 'w'	105 'i'	116 't'	104 'h'	32 ' '	67 'C'	97 'a'
0x402418:	110 'n'
```
We see a critical keyword Border, right?
Then we use strings command to find out the answer
```
$ strings bomb | grep Border
Border relations with Canada have never been better.
```
The first bomb is successfully defused.

## Step 2

