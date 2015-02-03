
Write Assembly code for required function of shellcode, string can be accessed by JMP -> CALL which will copy string in stack[1]

```
/* Code for creating a file and writing some content and closing file and exiting */
void main() {
__asm__(
"jmp l1;"
"l3: movl $0x8, %eax;"
"popl %ebx ;" 
"movl $0777, %ecx ;"  
"int $0x80;"			//create file
"movl %eax, %esi;"
"jmp l2;"
"l4: movl $9,%edx;" 
"popl %ecx;" 
"movl %eax, %ebx;"  
"movl $4, %eax;" 
"int $0x80;"  // write 
"movl $6, %eax ;" 
"movl %esi, %ebx ;"
"int $0x80;"	// close 
"movl $1, %eax ;"
"int $0x80 ;" // exit
"l1: call l3;"
".string \"foo.txt\";" 
"l2: call l4;			;"
".string \"You lose!\" ;"        
);
}
```

So, I copied hex representation using gdb.
(gdb) disas main
Dump of assembler code for function main:
   0x080483f0 <+0>:     push   %ebp
   0x080483f1 <+1>:     mov    %esp,%ebp
=> 0x080483f3 <+3>:     jmp    0x8048425 <l1>
   0x080483f5 <+5>:     mov    $0x8,%eax
   0x080483fa <+10>:    pop    %ebx
   0x080483fb <+11>:    mov    $0x1ff,%ecx
   0x08048400 <+16>:    int    $0x80
   0x08048402 <+18>:    mov    %eax,%esi
   0x08048404 <+20>:    jmp    0x8048432 <l2>
   0x08048406 <+0>:     mov    $0x9,%edx
   0x0804840b <+5>:     pop    %ecx
   0x0804840c <+6>:     mov    %eax,%ebx
   0x0804840e <+8>:     mov    $0x4,%eax
   0x08048413 <+13>:    int    $0x80
   0x08048415 <+15>:    mov    $0x6,%eax
   0x0804841a <+20>:    mov    %esi,%ebx
   0x0804841c <+22>:    int    $0x80
   0x0804841e <+24>:    mov    $0x1,%eax
   0x08048423 <+29>:    int    $0x80
   0x08048425 <+0>:     call   0x80483f5 <main+5>
   0x0804842a <+5>:     outsw  %ds:(%esi),(%dx)
   0x0804842c <+7>:     outsl  %ds:(%esi),(%dx)
   0x0804842d <+8>:     je,pn  0x80484a8 <__libc_csu_init+72>
   0x08048430 <+11>:    je     0x8048432 <l2>
   0x08048432 <+0>:     call   0x8048406 <l4>
   0x08048437 <+5>:     pop    %ecx
   0x08048438 <+6>:     outsl  %ds:(%esi),(%dx)
   0x08048439 <+7>:     jne    0x804845b
   0x0804843b <+9>:     insb   (%dx),%es:(%edi)
   0x0804843c <+10>:    outsl  %ds:(%esi),(%dx)
   0x0804843d <+11>:    jae    0x80484a4 <__libc_csu_init+68>
   0x0804843f <+13>:    and    %eax,(%eax)
   0x08048441 <+15>:    pop    %ebp
   0x08048442 <+16>:    ret
End of assembler dump.
(gdb) x/80bx main + 3
0x80483f3 <main+3>:     0xeb    0x30    0xb8    0x08    0x00    0x00    0x00    0x5b
0x80483fb <main+11>:    0xb9    0xff    0x01    0x00    0x00    0xcd    0x80    0x89
0x8048403 <main+19>:    0xc6    0xeb    0x2c    0xba    0x09    0x00    0x00    0x00
0x804840b <l4+5>:       0x59    0x89    0xc3    0xb8    0x04    0x00    0x00    0x00
0x8048413 <l4+13>:      0xcd    0x80    0xb8    0x06    0x00    0x00    0x00    0x89
0x804841b <l4+21>:      0xf3    0xcd    0x80    0xb8    0x01    0x00    0x00    0x00
0x8048423 <l4+29>:      0xcd    0x80    0xe8    0xcb    0xff    0xff    0xff    0x66
0x804842b <l1+6>:       0x6f    0x6f    0x2e    0x74    0x78    0x74    0x00    0xe8
0x8048433 <l2+1>:       0xcf    0xff    0xff    0xff    0x59    0x6f    0x75    0x20
0x804843b <l2+9>:       0x6c    0x6f    0x73    0x65    0x21    0x00    0x5d    0xc3
(gdb)
I created another C code to verify my shellcode. 
unsigned char shellcode[] = "\xeb\x30\xb8\x08\x00\x00\x00\x5b\xb9\xff\x01\x00\x00\xcd\x80\x89\xc6\xeb\x2c\xba\x09\x00\x00\x00\x59\x89\xc3\xb8\x04\x00\x00\x00\xcd\x80\xb8\x06\x00\x00\x00\x89\xf3\xcd\x80\xb8\x01\x00\x00\x00\xcd\x80\xe8\xcb\xff\xff\xff\x66\x6f\x6f\x2e\x74\x78\x74\x00\xe8\xcf\xff\xff\xff\x59\x6f\x75\x20\x6c\x6f\x73\x65\x21\x00";

int main(void) { ((void (*)())shellcode)(); }
It throws segmentation fault. 
-bash-4.1$ ./ss-c
Segmentation fault (core dumped)
Next, I tried gdb to debug the issue. 
Breakpoint 1, main () at ss-c.c:3
3       int main(void) { ((void (*)())shellcode)(); }
(gdb) si
0x080483fb      3       int main(void) { ((void (*)())shellcode)(); }
(gdb)
0x080496a0 in shellcode ()
(gdb) x/40i 0x080496a0
=> 0x80496a0 <shellcode>:       jmp    0x80496d2 <shellcode+50>
   0x80496a2 <shellcode+2>:     mov    $0x8,%eax
   0x80496a7 <shellcode+7>:     pop    %ebx
   0x80496a8 <shellcode+8>:     mov    $0x1ff,%ecx
   0x80496ad <shellcode+13>:    int    $0x80
   0x80496af <shellcode+15>:    mov    %eax,%esi
   0x80496b1 <shellcode+17>:    jmp    0x80496df <shellcode+63>
   0x80496b3 <shellcode+19>:    mov    $0x9,%edx
   0x80496b8 <shellcode+24>:    pop    %ecx
   0x80496b9 <shellcode+25>:    mov    %eax,%ebx
   0x80496bb <shellcode+27>:    mov    $0x4,%eax
   0x80496c0 <shellcode+32>:    int    $0x80
   0x80496c2 <shellcode+34>:    mov    $0x6,%eax
   0x80496c7 <shellcode+39>:    mov    %esi,%ebx
   0x80496c9 <shellcode+41>:    int    $0x80
   0x80496cb <shellcode+43>:    mov    $0x1,%eax
   0x80496d0 <shellcode+48>:    int    $0x80
   0x80496d2 <shellcode+50>:    call   0x80496a2 <shellcode+2>
   0x80496d7 <shellcode+55>:    outsw  %ds:(%esi),(%dx)
   0x80496d9 <shellcode+57>:    outsl  %ds:(%esi),(%dx)
   0x80496da <shellcode+58>:    je,pn  0x8049755
   0x80496dd <shellcode+61>:    je     0x80496df <shellcode+63>
   0x80496df <shellcode+63>:    call   0x80496b3 <shellcode+19>
   0x80496e4 <shellcode+68>:    pop    %ecx
   0x80496e5 <shellcode+69>:    outsl  %ds:(%esi),(%dx)
   0x80496e6 <shellcode+70>:    jne    0x8049708
   0x80496e8 <shellcode+72>:    insb   (%dx),%es:(%edi)
   0x80496e9 <shellcode+73>:    outsl  %ds:(%esi),(%dx)
   0x80496ea <shellcode+74>:    jae    0x8049751
   0x80496ec <shellcode+76>:    and    %eax,(%eax)
   0x80496ee <shellcode+78>:    add    %al,(%eax)
   0x80496f0 <completed.6257>:  add    %al,(%eax)
   0x80496f2:   add    %al,(%eax)
   0x80496f4 <dtor_idx.6259>:   add    %al,(%eax)
   0x80496f6 <dtor_idx.6259+2>: add    %al,(%eax)
   0x80496f8:   add    %al,(%eax)
   0x80496fa:   add    %al,(%eax)
   0x80496fc:   add    %al,(%eax)


gcc -fno-stack-protector -z execstack -o file_name file_name.c



References 
[1] http://insecure.org/stf/smashstack.html
[2]