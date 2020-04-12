## 创建
创建新文件时，新文件的用户ID就是进程的有效用户ID，组ID也是进程的组ID

## 运行时
在运行时，内核会对进程的所有读写执行操作进行检查。
- 如果进程的有效ID是Root，则无视文件的所有权限位
- 如果进程的有效ID是Owner，则检查对于Owner的读写执行的位是否OK
- 如果进程的有效ID是Grouper，则检查对于Grouper的读写执行的位是否OK
- 如果进程的有效ID是Other，则检查对于Other的读写执行的位是否OK

## 使用umask
使用这个函数或者shell命令可以设置默认的屏蔽位，比如shell中umask默认的掩码是0022，意味着默认关闭了文件的GROUP和OTHER的写入权限（RWX）