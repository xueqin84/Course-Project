1.make clean;
2.make -j5;



======================================================================

修改的文件有：
1。/usr/src/linux-2.6.31.12/kernel/sys.c
添加：
 asmlinkage int sys_mycall(char * infile, char *outfile){
	int infd, outfd, count;
	char buf[256];
	mm_segment_t fs;
	fs = get_fs();
	set_fs(get_ds());
	if((infd = sys_open(infile,O_RDONLY,0)) == -1)
		return -2;
	if((outfd = sys_open(outfile,O_WRONLY|O_CREAT, S_IRUSR|S_IWUSR)) == -1)
		return -3;
	while((count = sys_read(infd,buf,256))>0)
		if(sys_write(outfd,buf,count)!= count)
			return -5;
		if(count == -1)
			return -4;
	sys_close(infd);
	sys_close(outfd);
	set_fs(fs);
	return 0;
}
2。/usr/src/linux-2.6.31.12/arch/x86/include/unistd_32.h		 
添加＃define _NR_name nnn  
#define __NR_mycall 377 
貌似修改asm-x86和asm文件夹下面的都一样，最终都会修改到上面的unistd_32.h文件。
还有，也不存在修改总数的操作，直接添加一个你的那个号就可以了。
3。/usr/src/linux/arch/x86/kernel/syscall_table_32.S
添加 .long sys_mycall
应该是：.long SYSMBOL_NAME(sys_mysyscall)吧？
特别注意，前后的名称要一致。
4。make menuconfig
具体的参数不需要改变，只需要保存下，生成一个config文件就可以了，便于后面的bzImage的创建。
如何提示缺少Install ncurses (ncurses-devel) and try again.
这里提示说需要安装ncurses库文件。sudo apt-get install ncurses-dev 之后再make menuconfig试试！ 

如果要用make xconfig，则要先安装libqt3-compat-headers：执行 sudo apt-get install libqt3-compat-headers。 

5.make -j5
  make bzImage
  make modules
  make modules_install
  mkinitramfs -o /boot/initrd.img-mycall
  make install
修改：/boot/grub/grub.cfg
 增加如下部分：
menuentry "Ubuntu, Linux 2.6.31-12-mysys" {
        recordfail=1
        if [ -n ${have_grubenv} ]; then save_env recordfail; fi
	set quiet=1
	insmod ext2
	set root=(hd0,9)
	search --no-floppy --fs-uuid --set 8fc33daf-4470-4482-b265-6109f82d9dd2
	linux	/boot/vmlinuz-2.6.31.12 root=UUID=8fc33daf-4470-4482-b265-6109f82d9dd2 ro   quiet splash
	initrd	/boot/initrd.img-mycall
}

主要就是修改这这俩部分：
	linux	/boot/vmlinuz-2.6.31.12 root=UUID=8fc33daf-4470-4482-b265-6109f82d9dd2 ro   quiet splash
	initrd	/boot/initrd.img-mycall
为相应到参数就可以了。。

重启，进入相应到菜单后就可以了。
始终选择在root用户下面作，即使进入修改后到内核也是。

貌似修改一次sys.c后，不要make clean，也是可以在make -j5
这样的话，再次编译就少俩些步骤。快！！

成功！！！！！！！！！！！！！！！！！！！！
==============================================下删除===================
5.拷贝/usr/src/linux-2.6.27.45/arch/x86/boot/bzImage到/boot/bzImage
6。下面的已经实现了改变grub选项了，不需要手动修改grub了。
make clean  
make mrproper  //清除旧的配置文件
make xconfig（可视化选项）
make
make modules_install
make install 
还可以的是：
Make clean
Make bzImage
Make modules
Make modules_install
Make install


printk是：/var/log/messages.log


在debian下
aptitude install rar
rar -e *.rar

