# 用 Python 编写守护进程

在 Unix™ 和类似的系统中，守护进程 (daemon) 是一个后台程序，它不以终端或 X11 系统作为主要的输入输出。在其他操作系统中，守护进程也叫服务 (service)。一个守护进程通常执行一个专门的任务，比如 NTP 守护进程让你计算机的时间与 NTP 服务器保持同步。许多要执行异步任务的应用程序需要这类程序来让你生活得更轻松。例如说 Gearman 任务服务器的任务队列处理器就可以用这类程序来处理。

一个 Unix™ 下典型的守护进程，首先会关闭三个主要的输入输出流：_stdin_、_stdout_ 和 _stderr_，然后执行 `fork(2)` 系统调用，建立一个当前进程的镜像，一旦这一工作完成，父进程就会调用 `exit(1)` 系统调用，而子进程则在后台保持工作。由于 Python 的成为产品级语言的哲学思想，这个并不复杂的流程已经由 `daemon` 模块实现了，所以你可以用这个模块来实现一个守护进程程序。

	import os
	import sys
	import daemon
	import atexit
	
	def main():
		"""
		Main Program
		"""
		
		install_signal_handler()
		atexit.register(at_exit_handler)
		
		opts = parse_opts()
		config = parse_config(opts)
		if not opts.cwd:
			print("No Working Directory")
			sys.exit(0)
		
		with daemon.DaemonContext():
			os.chdir(opts.cwd)
			install_signal_handler()
			start_schedule(config, opts)
	
	""" Executes the main() Function """
	if __name__ == '__main__':
		main()

如果你观察一下代码，它使用 `os.chdir()` 函数修改了工作目录，因为一旦一个守护进程启动，它的工作目录常会自动修改到根目录或 `/`。一些守护进程如 Apache HTTP 服务器有一个预编译的工作目录，但它也允许使用 `chroot` 让其工作在其他目录下。此外，这个程序还配置了信号处理程序 (signal handler) 以允许程序来处理如 SIGHUP 和 SIGCHLD 这样的信号。`parse_opts()` 和 `parse_config()` 函数分别使用 `optparse` 和 `ConfigParser` 模块来解析守护进程的参数和配置。

`atexit` 模块用于保证大部分的程序资源——如文件、连接以及类似的东西——可以在守护进程中止时被释放或关闭。下面的 `atexit` 范例展示了释放资源的过程：

	from django.db import connections
	import atexit
	
	def at_exit_handler():
		"""
		At Exit Function (close all connections)
		"""
		for con in connections.all():
			try:
				con.close()
			except Exception, exc:
				log().error(exc)
		log().close()

信号处理程序之间有十分相似的行为，它注册一个用于处理信号的回调函数，这个函数会在每次一个信号接收到的时候被执行。比如说当 SIGHUP 信号到达的时候，我让守护进程更新配置而不是中止。

	import signal as sig
	import trackback as tb
	
	def rehash_daemon():
		"""
		Rehash the daemon, reading configurations again.
		"""
		global DAEMON_CONFIG
		DAEMON_CONFIG = parse_config()
	
	def install_signal_handler():
		"""
		Signal Handler Installer
		"""
		try:
			sig.signal(sig.SIGHUP, rehash_daemon)
		except Exception, exc:
			log().error(repr(exec))
			log().error(tb.format_exc(exc))
			return False

这让你可以创建一个健壮的守护进程：可以处理信号、在退出时资源可以被释放等等。记住作为一个有垃圾收集的语言，如果你没有正确地切断对象的引用，由于 GIL 循环的垃圾回收器使用的引用计数法，Python 会产生内存泄露。你的守护进程设计应该很好了。