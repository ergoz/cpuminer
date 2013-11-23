This is a multi-threaded CPU miner for Litecoin and Bitcoin,
fork of Jeff Garzik's reference cpuminer.

License: GPLv2.  See COPYING for details.

Downloads:  https://sourceforge.net/projects/cpuminer/files/
Git tree:   https://github.com/pooler/cpuminer

Dependencies:
	libcurl			http://curl.haxx.se/libcurl/
	jansson			http://www.digip.org/jansson/
		(jansson is included in-tree)

Basic *nix build instructions:
	./autogen.sh	# only needed if building from git repo
	./nomacro.pl	# only needed if building on Mac OS X or with Clang
	./configure CFLAGS="-O3"
	make

Notes for AIX users:
	* To build a 64-bit binary, export OBJECT_MODE=64
	* GNU-style long options are not supported, but are accessible
	  via configuration file

Basic Windows build instructions, using MinGW:
	Install MinGW and the MSYS Developer Tool Kit (http://www.mingw.org/)
		* Make sure you have mstcpip.h in MinGW\include
	If using MinGW-w64, install pthreads-w64
	Install libcurl devel (http://curl.haxx.se/download.html)
		* Make sure you have libcurl.m4 in MinGW\share\aclocal
		* Make sure you have curl-config in MinGW\bin
	In the MSYS shell, run:
		./autogen.sh	# only needed if building from git repo
		LIBCURL="-lcurldll" ./configure CFLAGS="-O3"
		make

Architecture-specific notes:
	ARM:	No runtime CPU detection. The miner can take advantage
		of some instructions specific to ARMv5E and later processors,
		but the decision whether to use them is made at compile time,
		based on compiler-defined macros.
		To use NEON instructions, add "-mfpu=neon" to CFLAGS.
	x86:	The miner checks for SSE2 instructions support at runtime,
		and uses them if they are available.
	x86-64:	The miner can take advantage of AVX, AVX2 and XOP instructions,
		but only if both the CPU and the operating system support them.
		    * Linux supports AVX starting from kernel version 2.6.30.
		    * FreeBSD supports AVX starting with 9.1-RELEASE.
		    * Mac OS X added AVX support in the 10.6.8 update.
		    * Windows supports AVX starting from Windows 7 SP1 and
		      Windows Server 2008 R2 SP1.
		The configure script outputs a warning if the assembler
		doesn't support some instruction sets. In that case, the miner
		can still be built, but unavailable optimizations are left off.

Usage instructions:  Run "minerd --help" to see options.

Connecting through a proxy:  Use the --proxy option.
To use a SOCKS proxy, add a socks4:// or socks5:// prefix to the proxy host.
Protocols socks4a and socks5h, allowing remote name resolving, are also
available since libcurl 7.18.0.
If no protocol is specified, the proxy is assumed to be a HTTP proxy.
When the --proxy option is not used, the program honors the http_proxy
and all_proxy environment variables.

Also many issues and FAQs are covered in the forum thread
dedicated to this program,
	https://bitcointalk.org/index.php?topic=55038.0
	
	



Building instructions
======================
Installing dependencies for building on Debian, Ubuntu and other APT-based distros:
Code:
```$ sudo apt-get install build-essential libcurl4-openssl-dev```
Installing dependencies for building on Fedora, RHEL, CentOS and other yum-based distros:
Code:
```$ sudo yum install gcc make curl-devel```
Installing dependencies for building on OpenSUSE and other ZYpp-based distros:
Code:
```$ sudo zypper in gcc make libcurl-devel```

Recipe for building on Linux:
Code:
```
$ wget http://sourceforge.net/projects/cpuminer/files/pooler-cpuminer-2.3.2.tar.gz
$ tar xzf pooler-cpuminer-*.tar.gz
$ cd cpuminer-*
$ ./configure CFLAGS="-O3"
$ make
```

Basic usage examples
===================
Code:
```
$ ./minerd --url=http://myminingpool.com:9332 --userpass=my.worker:password
$ ./minerd --url=stratum+tcp://myminingpool.com:3333 --userpass=my.worker:password
```
For more information:
Code:
```
$ ./minerd --help
```
FAQ / Troubleshooting

Q: Should I call this miner "cpuminer" or "minerd"?
A: The software package is called "cpuminer". "minerd" ("miner daemon") is just the name of the executable file provided by the package.

Q: My antivirus flags the Windows binary as malware.
A: That's a known false positive. More information here.

Q: When running configure I get the error "C compiler cannot create executables".
A: Make sure you typed CFLAGS="-O3" with a big O, not with a zero.

Q: autogen.sh dies with "error: possibly undefined macro: AC_MSG_ERROR".
Q: configure chokes on something like "LIBCURL_CHECK_CONFIG(, 7.10.1, ,'".
A: Make sure you have installed the development package for libcurl. If you have and you're still getting the error when compiling from git, try compiling from tarball instead.

Q: I'm trying to connect to a Stratum server, but I get "HTTP request failed: Empty reply from server".
A: Make sure you specified the correct protocol in the server URL (stratum+tcp://).

Q: Is there any command-line option I can play with to make it mine faster?
A: No. The miner automatically picks the best settings for the CPU it is run on.

Q: What's the difference between the two available algorithms, scrypt and sha256d?
A: They are completely different proof-of-work algorithms. You must use scrypt for Litecoin, and you must use sha256d for Bitcoin. The default algorithm is scrypt, so for Bitcoin mining you have to specify --algo=sha256d.

Q: Will this miner use a lot of RAM when using the scrypt algorithm?
A: No, that's a GPU thing.

Q: How do I make the miner write its output to a file instead of printing it to the screen?
A: Just redirect the standard error stream to file:
Code:
minerd [OPTIONS] 2> myfile
You may also want to use the --quiet/-q option to disable the per-thread hashmeter.
On *nix, you probably also want to use the --background/-B option to fork in the background.


Original post (December 19, 2011) follows. Please note that most of the technical details are now outdated.
I have recently rewritten the heart of the scrypt hashing function used by the jgarzik/ArtForz cpuminer in assembly language, to see if this could bring some more speed. Apparently it did. Smiley
The source code is now available at GitHub:
https://github.com/pooler/cpuminer
The build process for Linux should be the same as before.

In the new code I tried to take full advantage of SSE2 instructions, which are available since the Pentium 4. Unfortunately, AMD's implementation of these instructions is not as fast as Intel's... well, ok, sadly it's nearly two times slower. For this reason, I had to write separate versions of the hashing functions. You don't need to worry about this, though, since the new function should be able to auto-detect your cpu and automatically select the best algorithm.

Long polling patch
This release also includes a new --timeout option that I originally added to solve a problem with long polling. Apparently the LP thread doesn't behave nicely under certain network conditions, as reported by various users. So, if you experienced high stale rates with the previous miner, you should definitely try out this new version.
Many thanks to SockPuppet, aka shawnp0wers, who helped me nail down the issue!

Some Technical Details
The current release includes four different implementations of the scrypt core, each one designed for a different hardware.
A fallback plain x86 version, to be used when SSE2 instructions are not available (Pentium III, Athlon XP and earlier processors).
A 32-bit version using SSE2, for use on the Pentium 4, Pentium M, Core, Atom, plus all 64-bit cpus running in a 32-bit OS.
A 64-bit version for Intel processors, i.e. Core 2, i3, i5, i7. This version can in most cases double the speed of the previous miner.
A 64-bit version for AMD processors, i.e. Athlon 64, Phenom, Sempron and the like. The speed increase here can range from 5% to 80%.
The first two versions only get compiled in the 32-bit miner, the last two only in the 64-bit miner. The miner uses the CPUID instruction to choose which version to use.

Compiler Flags
One cool aspect of assembly code is that users no more need to play with compiler flags to get the best performance. Configuring the build with just CFLAGS="-O3" is now more than enough to get efficient code. This also means that we no more need separate specialized binaries for Intel and AMD cpus. Just a 32-bit and a 64-bit version.

Personal Notes
Someone on IRC asked me why I am releasing this miner, instead of keeping it for myself or for my pool. Well, that's exactly the point. It is important for Litecoin that everybody has access to the most efficient mining software!
Someone might worry about the effect of this release on market prices, but consider this: if everybody starts using the new miner, the hash rate will go up, but so will difficulty, so nothing will ultimately change. I actually think this new miner will be very beneficial to Litecoin, because it should make mining easier for beginners (see compiler flags).
As crazy_rabbit wrote in another thread, one big plus of Litecoin is that everybody can participate. Well, consider this: now you can effectively mine on an Atom! Smiley

Alright folks... I hope you enjoy the performance boost. Consider this as my Christmas present to the community!
