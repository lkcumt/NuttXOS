NuttX 操作系统用户手册
============
__原作者：__By Gregory Nutt

__Last Updated:__ September 28, 2013

__翻译：__cumtlk

__版本：__V1.0

1.0 介绍
-------------

本文档从固件开者的角度提供了NuttX 实时操作系统一般的使用信息。
### 1.1 文档概述
本用户手册分为三个部分，外加一个索引：
* __部分1.0，介绍：__本节提供了NuttX用户手册的概述。
* __部分2.0，操作系统接口：__本节详细介绍了NuttX提供的程序接口。本节被划分为几个段落来描述操作系统接口的不同部分。
* __部分3.0，操作系统数据结构：__本节介绍了在NuttX接口中使用的数据结构。

### 1.2 目标读者与使用范围
目标读者是在NuttX上开发应用的固件开发者。具体来说，这个文档只是在说明应用开发者可以使用的NuttX实时操作系统的API。因此，这个文档并没有集中在NuttX的组织或实现的任何技术细节。那些技术细节在[NuttX Porting Guide](http://nuttx.org/Documentation/NuttxPortingGuide.html)中提供。应用开发者也需要关于配置和编译NuttX的信息。那些信息也可以在[NuttX  Porting  Guide](http://nuttx.org/Documentation/NuttxPortingGuide.html)中找到。

2.0 操作系统接口
----------------
本节介绍了NuttX操作系统每一个C可调用的接口。

__函数原型：__提供接口函数的C原型。

__说明：__讨论了接口函数执行的操作。

__输入参数：__列出了所有的输入参数，并对每个参数作了简要的介绍。

__返回值：__接口函数返回的每一个可能的值都列了出来。返回值作为副作用（通过指针输入参数或者全局编领）将在接口函数的描述中加以声明。

__假设与限制：__通过接口函数产生的任何不寻常的假设或者使用接口函数的任何非显而易见的局限性将会在这里说明。

__POSIX相容性：__NuttX接口与它相应的POSIX接口之间的任何重要的差异都会在这里指出。

__注：__为了实现NuttX接口函数的独立的名称空间，函数名和类型之间的差异可以预期的，在这些段落中不会被定义为差异。
### 2.1 任务控制接口
__任务：__NuttX是一个平面地址操作系统，因此，它不支持这样的进程，比如说linux中的方式。NuttX仅仅支持运行在相同地址空间的简单的线程。然而，编程模型使任务和pthreads之间有区别。
* 任务是具有一定程度独立性的线程。
* Pthreads 共享某些资源。

__文件描述符和流：__这尤其适应于，在打开的文件描述符和数据流区域。当一个任务使用本节的接口启动时，它将会被创建，并最多有三个文件打开。

如果CONFIG_DEV_CONSOLE被定义，前三个文件描述符（对应标准输入、标准输出、标准错误）将会被复制给新任务。由于这些文件的文件描述符被复制。一旦这些文件描述符被复制，孩子任务可以足有关闭它们或者以任何方式操作它们，而不会影响父任务。在一个任务中，与文件相关的操作（打开、关闭等等）不会对其他任务造成影响。一旦这三个文件描述符被复制了，它也有可能进行某种程度的重定向。

pthreads，另一方面，将始终与符线程共享文件描述符。在这种情况下，文件操作将会只影响所有从同一父线程启动的线程。

__执行文件系统中的程序：__NuttX还为驻留在文件系统单独建立程序的执行提供了内部接口。然而，这些内部接口是非标准的，而且在NuttX二进制加载器和NXFLAT被文档化。

__任务控制接口：__下面的任务控制接口就是由NuttX提供的：

灵感来自于VxWorks接口非标准任务控制接口：

* 2.1.1 task_create
* 2.1.2 task_init
* 2.1.3 task_activate
* 2.1.4 task_delete
* 2.1.5 task_restart

标准接口：

* 2.1.6 exit
* 2.1.7 getpid

标准的vfork和exec[v|1]接口：

* 2.1.8 vfork
* 2.1.9 execv
* 2.1.10 execl

标准的posix_spawn 接口：

* 2.1.11 posix_spawn and posix_spawnp
* 2.1.12 posix_spawn_file_actions_init
* 2.1.13 posix_spawn_file_actions_destroy
* 2.1.14 posix_spawn_file_actions_addclose
* 2.1.15 posix_spawn_file_actions_adddup2
* 2.1.16 posix_spawn_file_actions_addopen
* 2.1.17 posix_spawnattr_init
* 2.1.18 posix_spawnattr_getflags
* 2.1.19 posix_spawnattr_getschedparam
* 2.1.20 posix_spawnattr_getschedpolicy
* 2.1.21 posix_spawnattr_getsigmask
* 2.1.22 posix_spawnattr_setflags
* 2.1.23 posix_spawnattr_setschedparam
* 2.1.24 posix_spawnattr_setschedpolicy
* 2.1.25 posix_spawnattr_setsigmask

灵感来自于posix_spawn的非标准任务控制接口：

* 2.1.26 task_spawn
* 2.1.27 task_spawnattr_getstacksize
* 2.1.28 task_spawnattr_setstacksize

### 2.1.1 task_create

__函数原型：__

	#include <sched.h>
	int task_create(char *name, int priority, int stack_size, main_t entry, char * const argv[]);

__描述：__该函数创建和激活一个新的任务，该任务带有一个指定的优先级，返回系统赋予的ID。入口地址的入口是任务的“main”函数的地址。一旦c环境被建立了，该函数就会被调用。指定的函数将会被调用，并带有四个参数。指定的路径返回，exit()的调用将会自动进行。

__输入参数：__
* name. 新任务的名字。
* priority. 新任务的优先级。
* stack_size. 需要的栈的大小（字节）。
* entry. 新任务的入口点。
* argv. 指向输入参数数组的指针。最多可以提供CONFIG_MAX_TASK_ARG个参数。如果传递的参数少于CONFIG_MAX_TASK_ARG，参数列表应以NULL argv[]值被终止，如果不需要参数，argv可以为NULL。

__返回值：__

* 返回新任务的非零任务ID，或者返回ERROR，如果内存不足或者任务无法创建(错误没有设置)。

__假设/限制：__

__POSIX 兼容性：__这是个NON-POSIX接口。VxWorks提供下面相似的接口：

	int taskSpawn(char *name, int priority, int options, int stackSize, FUNCPTR entryPt,
	int arg1, int arg2, int arg3, int arg4, int arg5,
	int arg6, int arg7, int arg8, int arg9, int arg10);

NuttX的task_create()与VxWorks的taskSpawn()不同，主要有一下几点：

* 接口名字。
* 参数类型不同。
* 没有任何选项参数。
* 很多参数可以传给一个任务（VxWorks 支持10个）。

## 2.1.2 task_init

__函数原型：__

	#include <sched.h>
	int task_init(struct tcb_s *tcb, char *name, int priority, uint32_t *stack, uint32_t stack_size,
	maint_t entry, char * const argv[]);

__描述：__

该函数初始化一个任务控制块（TCB），准备开始一个新的线程。它执行task_create()功能上的一个子集（见上文）。

不像task_create()，task_init()不会激活任务。任务的激活必须通过调用task_activate()。

__输入参数：__

* tcb. 新任务TCB的地址。Address of the new task's TCB
* name. 新任务的名字（未使用）。Name of the new task (not used)
* priority. 新任务的优先级。Priority of the new task
* stack. 预分配的栈的开始。Start of the pre-allocated stack
* stack_size. 预分配的栈的大小（字节）。size (in bytes) of the pre-allocated stack
* entry. 新任务的入口点。Entry point of a new task
* argv. 指向输入参数的数组的指针。最多可以提供CONFIG_MAX_TASK_ARG个参数。如果传递的参数少于CONFIG_MAX_TASK_ARG，参数列表应以NULL argv[]值被终止，如果不需要参数，argv可以为NULL。

__返回值：__

* OK，或者ERROR，如果任务不能被初始化。

这个函数仅当它不能分配一个新的、唯一的任务ID给TCB时（错误没有设置），会失败。

__假设/限制：__

* 提供task_init()来支持内部操作系统的功能。不推荐正常使用时用它。task_create()是初始化和开始一个新任务的首选机制。
 
__POSIX 兼容性：__这是NON-POSIX的接口。VxWorks提供以下相似的接口：

	STATUS taskInit(WIND_TCB *pTcb, char *name, int priority, int options, uint32_t *pStackBase, int stackSize, 
	FUNCPTR entryPt, int arg1, int arg2, int arg3, int arg4, int arg5, 
	int arg6, int arg7, int arg8, int arg9, int arg10);

NuttX task_init()与VxWorks的taskInit()有以下几点不同：

* 接口名字。
* 参数类型或者参数不同。
* 没有选项参数。
* 很多参数可以被传给一个任务（VxWorks 支持10）。

### 2.1.3 task_activate

__函数原型：__

	#include <sched.h>
	int task_activate(struct tcb_s *tcb);

__描述：__该函数激活被task_init()创建的任务。没有激活，一个任务不会被调度执行。

__输入参数：__

* tcb. 任务的TCB（与task_init 参数相同）。

__返回值：__

* OK，或者ERROR，如果任务不能被激活（errno 不被设置）。

__假设/限制：__

* 提供task_activate()是来支持内部OS功能。不推荐正常使用。task_create()是推荐的初始化和开始一个新任务的机制。  

__POSIX 兼容性：__这是NON-POSIX接口。VxWorks提供以下类似的接口：

	STATUS taskActivate(int tid);

NuttX task_activate()与VxWorks的taskActivate()有以下不同：

* 函数名字
* VxWorks的taskActivate，pid参数应该指向WIND_TCB强制转化为整数。

### 2.1.4 task_delete

__函数原型：__

	#include <sched.h>
	int task_delete(pid_t pid);	

__描述：__

此函数使指定的任务终止退出。--其栈和TCB将会被释放。这个函数与task_create()相对应。

__输入参数：__

* pid. 要删除的任务的任务ID。ID是0表示正在调用的任务。由正在调用的任务发出的任何尝试都将被自动重定向到exit()。

__返回值：__

* OK，或者ERROR，如果任务不能被删除。该函数会失败，如果提供的pid与任务不一致的话（errno 没有设置）。

__假设/限制：__

task_delete()使用时必须注意：如果任务拥有资源（例如，已分配的内存或者被其他任务需要的信号），那么，task_delete()可以搁浅那些资源。

__POSIX 兼容性：__这是NON-POSIX接口。VxWorks提供以下类似的接口：
	
	STATUS taskDelete(int tid);

NuttX的task_delete()与VxWorks的taskDelete()有以下不同点：

* 不支持提供任务删除程序（因为VxWorks taskDeleteHookAdd()不支持）。然而，如果atexit()或者on_exit支持使能了，当任务删除的时候，那些将会被调用。
* 支持删除自己，仅仅因为task_delete()将重新直接处理到exit()。

### 2.1.5 task_restart

__函数原型：__

	#include <sched.h>
	int task_restart(pid_t pid);

__描述：__该函数“重新启动”一个任务。任务首先被终止，然后重新初始化，以相同的ID、优先级、原来的入口点、栈大小、和当它第一次启动时它拥有的参数。

__注意：__

1. 不进行正常任务退出清理。比如，文件描述符不会被关闭。任何在重新启动之前打开的文件都将在任务重启之后保持打开。
2. 在任务重新启动之前已经被任务分配的内存不会被释放。一个需要重新启动的任务必须被设计成以这样的方式来避免内存泄漏。
3. 已经初始化的数据不会重置。所有的全局或者静态变量都会保持与任务被终止时的一样的状态。比如，这个功能可用于重新启动的任务来检测它已经被重新启动。

__输入参数：__

* pid.要删除的任务的任务ID。ID为0表示调用任务（然而，支持一个任务重新启动自己还没有被实现）。

__返回值：__

* OK，或者ERROR，如果任务ID非法或者任务不能被重新启动。该函数会失如果：（1）PID为0或者调用任务的pid被提供（功能未实现）；（2）pid不与任何系统已知的任务相关联。

__假设/限制：__

__POSIX 兼容性：__这是NON-POSIX接口。VxWorks提供如下类似的接口：

	STATUS taskRestart (int tid);

NuttX的task_restart()与VxWorks的taskRestart()有以下不同：

* NuttX不支持重新启动当前正在运行的任务。
* VxWorks描述中提到了ID、优先级等等。当任务被终止时，采取他们过去有的值。

## 2.1.6 exit

__函数原型：__

	#include <sched.h>
	void exit(int code);
	
	#include <nuttx/unistd.h>
	void _exit(int code);

__描述：__该函数使调用函数退出--它的栈和TCB将会被释放。exit 会冲洗流、关闭文件描述符，并且执行任何用atexit()或者on_exit()注册的函数，这是exit与_exit不同的地方。

__输入参数：__

* code. （忽略）。

__返回值：__没有返回值。

__假设/限制：__

__POSIX 兼容性：__与ANSI接口等价。

	void exit(int code);

UNIX接口：

	void _exit(int code);

NUttX exit()与ANSI exit()有以下不同：

* code参数被忽略。

### 2.1.7 getpid

__函数原型：__

	#include <unistd.h>
	pid_t getpid(void);

__描述：__该函数返回调用任务的任务ID.如果在中断级别调用，任务ID将会无效。

__输入参数：__无。

__返回值：__

* 调用任务的任务ID。

__假设/限制：__

__POSIX 兼容性：__兼容POSIX接口，名字相同。

### 2.1.8 vfork

	#include <unistd.h>
	pid_t vfork(void);

__返回值：__一旦成功完成，vfork()返回0给子进程，并且返回子进程的进程ID给父进程。否则，返回-1给父进程，无子进程创建，errno被设置表示这个错误。

### 2.1.9 execv

	#include <unistd.h>
	#ifdef CONFIG_LIBC_EXECFUNCS
	int execv(FAR const char *path, FAR char *const argv[]);
	#endif
__描述：__NuttX不支持进程，这个函数蛮复杂的，具体用到的话，看英文详细解释吧。。

__输入参数：__

* path: 被执行的程序的路径。如果CONFIG_BINFMT_EXEPATH在配置中被定义了，那么这可能是一个相对的路径，相对于当前工作目录。否则，path必须是绝对路径。
* argv: 指针数组，数组元素是指针，指针指向字符串参数。数组结尾以NULL结束。

### 2.1.10 execl

	#include <unistd.h>
	#ifdef CONFIG_LIBC_EXECFUNCS
	int execl(FAR const char *path, ...);
	#endif
__描述：__功能类似execv()，都是exce家族的，用到的话看英文详细。

__输入参数：__

* path：路径。如果CONFIG_BINFMT_EXEPATH在配置中定义，则是相对路径。否则，path是绝对路径。
* ...:程序接收的string参数列表。0表示列表的末尾。

__返回值：__不会成功返回的。失败的话，会返回-1并且会设置errno值。

### 2.1.11 posix_spawn and posix_spawnp

	#include <spawn.h>
	int posix_spawn(FAR pid_t *pid, FAR const char *path,
	FAR const posix_spawn_file_actions_t *file_actions,
	FAR const posix_spawnattr_t *attr,
	FAR char *const argv[], FAR char *const envp[]);
	int posix_spawnp(FAR pid_t *pid, FAR const char *file,
	FAR const posix_spawn_file_actions_t *file_actions,
	FAR const posix_spawnattr_t *attr,
	FAR char *const argv[], FAR char *const envp[]);
__描述：__这两个函数将创建一个新的有普通可执行文件创建的孩子任务。

__输入参数：__

* pid:
* path or file:
* file_actions:
* attr:
* argv:
* envp:

__返回值：__
