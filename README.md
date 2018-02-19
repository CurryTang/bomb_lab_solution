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
Firstly, let's have a look at the code
``` asm
0000000000400efc <phase_2>:
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq  
```
Having a look at the code structure, you should notice that there exists a loop structure. What' more, there's a function call to read_six_numbers(), we can inspect it
``` asm
000000000040145c <read_six_numbers>:
  40145c:	48 83 ec 18          	sub    $0x18,%rsp
  401460:	48 89 f2             	mov    %rsi,%rdx
  401463:	48 8d 4e 04          	lea    0x4(%rsi),%rcx
  401467:	48 8d 46 14          	lea    0x14(%rsi),%rax
  40146b:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  401470:	48 8d 46 10          	lea    0x10(%rsi),%rax
  401474:	48 89 04 24          	mov    %rax,(%rsp)
  401478:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  40147c:	4c 8d 46 08          	lea    0x8(%rsi),%r8
  401480:	be c3 25 40 00       	mov    $0x4025c3,%esi
  401485:	b8 00 00 00 00       	mov    $0x0,%eax
  40148a:	e8 61 f7 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  40148f:	83 f8 05             	cmp    $0x5,%eax
  401492:	7f 05                	jg     401499 <read_six_numbers+0x3d>
  401494:	e8 a1 ff ff ff       	callq  40143a <explode_bomb>
  401499:	48 83 c4 18          	add    $0x18,%rsp
  40149d:	c3                   	retq 
```
Up till now, you should be able to find out that in this part, we are required to enter six numbers. Considering this line of code 
``` asm
400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
```
It's obvious that the first number should be 1. We can find the latter numbers from the loop structure. 
``` asm
400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
```
We multiply the number by 2 each step, so we guess the sequence to be 1, 2, 4, 8, 16, 32, which is the answer.

## Step 3

