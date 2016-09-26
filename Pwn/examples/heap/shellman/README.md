##Tips
1.��Ϊ������alarm����ʱ��60s
handle SIGALRM print nopass ����������alarm�ص�(ʵ������Ҳû��gdb����, ��֪��Ϊʲô)
2.������Ƚϸ���(���ͻ��˺þ�,����Ҳ�������ر�����), ��ͼ�������΢�õ�

##�ṹ��
 sh_st           struc ; (sizeof=0x18, mappedto_1) ; XREF: .bss:stru_6016C0/r
00000000 flag            dq ?                    ; XREF: new_sh+AA/w
00000008 size            dq ?                    ; XREF: new_sh+B5/w
00000010 name_ptr        dq ?                    ; XREF: new_sh+BC/w
00000018 sh_st           ends

##©����
get_name(sh_st->name_ptr, length); // ����lengthҲ������ģ���������û�ж�length��飬������new������size���С��������ͻᷢ�������

##����ԭ��
small chunk // large chunk ��ĳ���ڵ�chunk free֮��, ��chunk��ǰ���chunk���, ���Ϊfree, �ͽ�����ǰ�����α�, ���ҰѼ�鵽��free chunk��bins˫������ɾ��
��linux�ѵĽṹ����֪�������������chunk1���ٴ���chunk2(�������˵��name_ptrָ��ĵط�)��chunk��ͷ����Ϣ������
0x00 pre chunk size
0x08 chunk size |N|M|P| P=0 : pre chunk free
0x10 fd
0x18 bk
������ҪP��ʾǰһ�ѿ��Ƿ�ʹ��
�����P���ǳ�0�Ļ����ͻ�ʹ��������ִ��ulink, ���ҳ���������Ϊname_ptr_0 ��free chunk ��������Ϊname_ptr_0ָ����bin���һ����
```
/* Take a chunk off a bin list */
#define unlink(AV, P, BK, FD)
{                             
    FD = P->fd;	//P == &chunk1					          
    BK = P->bk;								   
    if (__builtin_expect (FD->bk != P || BK->fd != P, 0))		    
      malloc_printerr (check_action, "corrupted double-linked list", P, AV); 
    else
    {								    
        FD->bk = BK;				
        BK->fd = FD;                //��Ҫע������ ���ǰ�free chunk��˫����ɾ��
      
        //large chunk do something
        //...
    }
}
```
P->fd->bk = P->bk; 
P->fd = addr-0x18 , P->bk = value ʱ 
P->fd->bk = P->bk �൱���޸�*((addr-0x18)+0x18) Ϊ value

##����
��Ҫ������:
1.��������chunk chunk[0~3]
2.edit(chunk0) ����chunk0Ϊfree chunk���Ҹ�����chunk1��ͷ��
3.Ȼ���ͷ�chunk1 ����unlink �޸�0x6016d0(name_ptr 0�ĵ�ַ)Ϊǰ���һ��ַ(0x6016b8)
4.����edit(chunk0) ��0x6016b8��һ�οռ��ֵ����, ���Ҹպ��ܹ����ĵ�name_ptr 0�ĵ�ַ(����Ϊgot����free�ĵ�ַ)
5.����list(0)�Ϳ���leak����ʵ��free��ַ, Ȼ�������ʵ��system������ַ, �ƹ�ASLR
6.�ٴε���edit(0) ��got����free����, ����Ϊ4�л�õ�system������ַ, ��ô֮��ֻҪ����free, ���൱�ڵ���system
7.ֻҪ֮ǰ��chunk3�Ǳ�д��"/bin/sh"����ַ�����free��, ����ִ��system��
8.��һ�θ�orz ���ű��˵��Լ����ĸ�(��)����

###����system������ַ
����libc���е�free������ַ
objdump -T lib.so.6 | grep system 
������so��: lib_sys_addr =  0x0000000000046590
            lib_free_addr = 0x0000000000082d00
�ṩ��so��: lib_sys_addr =  0x0000000000046640
            lib_free_addr = 0x0000000000082df0
system_addr = free_addr - lib_free_addr + lib_sys_addr

###�����Ŀ�
1.data_0  = 'p'*(first_size-0x20)   ����α���ʱ������������Ҫ��ȥͷ����Ϣ, ������newʱ��Ĵ�С
2.size_1  = p64(second_size + 0x10) ����Ҫע��ڶ�����Ĵ�С�Ǽ���ͷ����pre chunk size������chunk size ��û��fd��bk
3.��ȡleak����free��ַ
�����õ���zio �����Լ���pwn(û������)
������ǿӲ���, ���˺ü���Сʱ(��֪��Ϊʲôû������), ������Ƕ�ȡ��ʱ��û�аѵ�ַת����orz(�Һò˰�)
���õ�ת�������ñ�orz ��ʵ��򵥵�����zio��l64
return l64(io.read(16).decode('hex'))
```
recv_addr = p.read(8 * 2) #����ע��Զ�ȡ�ĵ�ַ�����ʵ���ת��
ret_addr = ''
for i in range(8):
    ret_addr += recv_addr[2*i+1] + recv_addr[2*i]
print type(recv_addr), len(recv_addr), recv_addr
print type(ret_addr), len(ret_addr), ret_addr
return int(ret_addr[::-1], 16)
```

##Reference
[Linex ��](http://tyrande000.how/2016/02/20/linux%E4%B8%8B%E7%9A%84%E5%A0%86%E7%AE%A1%E7%90%86/)
[linux �� ��](https://jiji262.github.io/wooyun_articles/drops/Linux%E5%A0%86%E7%AE%A1%E7%90%86%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20(%E4%B8%8A%E5%8D%8A%E9%83%A8).html)
[linux �� ��](https://jiji262.github.io/wooyun_articles/drops/Linux%E5%A0%86%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90(%E4%B8%8B%E5%8D%8A%E9%83%A8).html)
[linux �� unlink����](http://tyrande000.how/2016/03/21/linux%E5%A0%86%E6%BA%A2%E5%87%BA%E5%AE%9E%E4%BE%8B%E5%88%86%E6%9E%90/)
ǿ���������簲ȫ��ս��WriteUp
