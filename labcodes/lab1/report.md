#Ucore lab- lab1ʵ�鱨��
##��ϰ1
###1.	����ϵͳ�����ļ�ucore.img�����һ��һ�����ɵģ�
����make v=�ɵ�
+ cc kern/init/init.c  //������������bin/kernel������ļ�
+ cc kern/libs/stdio.c
+ cc kern/libs/readline.c
+ cc kern/debug/panic.c
+ cc kern/debug/kdebug.c
+ cc kern/debug/kmonitor.c
+ cc kern/driver/clock.c
+ cc kern/driver/console.c
+ cc kern/driver/picirq.c
+ cc kern/driver/intr.c
+ cc kern/trap/trap.c
+ cc kern/trap/vectors.S
+ cc kern/trap/trapentry.S
+ cc kern/mm/pmm.c
+ cc libs/string.c
+ cc libs/printfmt.c
+ ld bin/kernel   //��������bin/kernel
+ cc boot/bootasm.S  //����bootasm.S  bootmain.c  sign.c
+ cc boot/bootmain.c
+ cc tools/sign.c
+ ld bin/bootblock
'obj/bootblock.out' size: 488 bytes
build 512 bytes boot sector: 'bin/bootblock' success!
��¼��10000+0 �Ķ���
��¼��10000+0 ��д��
5120000 bytes (5,1 MB, 4,9 MiB) copied, 0,0327003 s, 157 MB/s
��¼��1+0 �Ķ���
��¼��1+0 ��д��
512 bytes copied, 0,000196915 s, 2,6 MB/s
��¼��146+1 �Ķ���
��¼��146+1 ��д��
74816 bytes (75 kB, 73 KiB) copied, 0,00037946 s, 197 MB/s

���Ͽɵã��ȱ�����������bin/kernel������ļ���Ȼ����������bin/kernel��
Ȼ�����bootasm.S  bootmain.c  sign.c ���ٸ���sign�淶����obj/bootblock.o
�������ucore.img��

###2.һ����ϵͳ��Ϊ�Ƿ��Ϲ淶��Ӳ��������������������ʲô��
��һ����������������ֻ��512�ֽ�
�ڶ���������������ֽ�Ϊ0x55��0xAA
�������ɲ�����466�ֽڵ���������Ͳ�����64�ֽڵ�Ӳ�̷�������������ֽڵĽ��������

##��ϰ2
###1.��CPU�ӵ��ִ�еĵ�һ��ָ�ʼ����������BIOS��ִ��
�޸�lab1/tools/gdbinit�������Ϊ
```
set architecture i8086
target remote :1234
define hook-stop
x/i $pc
end
```
Ȼ����lab1����make debug
ʹ��si����ɽ��е�������
###2.�ڳ�ʼ��λ��0x7c00����ʵ��ַ�ϵ�,���Զϵ�����
�޸�ͬ�����ļ�gdbinitΪ
```
file obj/bootblock.o
set architecture i8086
target remote :1234
b *0x7c00
continue
```
���
###3.��0x7c00��ʼ���ٴ�������,���������ٷ����õ��Ĵ�����bootasm.S�� bootblock.asm���бȽ�
�������ͬ

###4.�Լ���һ��bootloader���ں��еĴ���λ�ã����öϵ㲢���в���
�ڽ��뱣��ģʽ��λ�����öϵ�,���е��ϵ���ֹͣ
��gdbinit�����޸�
```
file obj/bootblock.o
set architecture i8086
target remote :1234
b protcseg
continue
```

##��ϰ3
###����bootloader���뱣��ģʽ�Ĺ���
bootbloader�������������ж�,֮�󽫶μĴ�������,��A20��ַ��,����GDT�Ļ���ַ,�л�������ģʽ,��ת��32λ����. ��32λ������, bootloader�������ñ���ģʽ�µĶμĴ���, Ȼ������ջ��ָ��, ֮����ת��C���롣

##��ϰ4
###����bootloader����ELF��ʽ��OS�Ĺ���
�ȶ�ȡelf�ļ�ͷ,��֤�ļ�ͷ�е�magic number,��ȡ����ͷ������һ��program header�ṹ�����飬���л�ó����������Ϣ���Ҷ�ȡ��ָ���ĵ�ַ��,����תִ�и��ļ�
�����ʾ���õ�ַǿ��ת����һ������Ϊ�յĺ�������ִ�иú���
```
  ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))()
```

##��ϰ5
###ʵ�ֺ������ö�ջ���ٺ���
���ȣ�����ͨ��read_ebp()��read_eip()��������ȡ��ǰebp�Ĵ�����eip �Ĵ�������Ϣ��
Ȼ�������ǰջ֡���ݣ�Ȼ���ٻ��ǰһ��ջ֡��ebp��eip(�õ�ǰ��ebpֵ��Ϊ��ַ)��֮���ٽ��еݹ鵽����ջ֡�������ʡ�
ִ�� make qemu �õ�������������һ���ǣ�
```
ebp:00007bf8 eip:00007d6e args:c031fcfa c08ed88e 64e4d08e fa7502a8
    <unknow>: -- 0x00007d6d --
```
���Ӧ���ǵ�һ��ʹ�ö�ջ�ĺ�����bootmain.c�е�bootmain��bootloader���õĶ�ջ��0x7c00��ʼ��ʹ�á�call bootmain��ת��bootmain������callָ��ѹջ������bootmain��ebpΪ0x7bf8��

##��ϰ6
###�����жϳ�ʼ���ʹ���
####1.�ж���������Ҳ�ɼ��Ϊ����ģʽ�µ��ж���������һ������ռ�����ֽڣ������� ��λ�����жϴ���������ڣ�
һ������ռ8���ֽڣ�����0-1λ��6-7λ��������ʾ��ַ��2-3λΪ��ѡ���ӡ���ѡ���Ӵ�GDT����LDT��ȡ�û�ַ����Ӻ�õ��������
####2.��������kern/trap/trap.c�ж��ж���������г�ʼ���ĺ���idt_init
������������__vectors������洢��ÿ���жϴ�������ƫ�ƣ�Ȼ��idt��ÿһ����SETGATE�����ã�������lidt()��������
####3.��������trap.c�е��жϴ�����trap
����һ�����������ۼӣ�ÿ���ۼƵ�100 ticksʱ��ӡ�����



