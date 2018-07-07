---
title: "Debugging unresponsive Ruby Applications with gdb"
date: 2018-06-22T07:56:47+10:00
categories: ['Linux', 'Ruby']
tags: ['ruby', 'gdb', 'debugging']
draft: false
---

Every once in a while, I encounter random freezing/hanging when developing Ruby applications and often find myself having to Google to find the correct gdb commands to use to debug these sorts of issues. To make life easier for myself (and hopefully for others out there), I've decided to document them here for future reference. I will (hopefully!) add to this page as I come across new strategies for debugging these sorts of issues.

Firstly, this guide assumes that you have access to a Unix machine with the GNU debugger ([gdb](https://www.gnu.org/s/gdb/)) installed and that you're running plain-old Ruby (MRI/YARV/KRI) - as the methods described on this page rely on the underlying C methods that the Ruby VM calls.

 If you're following along and don't have a "frozen" Ruby process, you can simulate a frozen/hung process by running a Ruby script that sleeps, such as:

```ruby
#!/usr/bin/env ruby
def perform_some_cool_function 
  sleep 20
end

until false do
  perform_some_cool_function
end
```

The first thing we will need to do is attach gdb to the frozen/hung process. To do this, we will need to get the process ID (PID) of our frozen application and then fire up gdb.
  ```bash
  $ ps -ef | grep my-ruby-script
  username   18198  0.6  0.0 149456  9864 pts/0    Sl+  17:47   0:00 ruby my-ruby-script.rb
  username   18973  0.0  0.0 119464   968 pts/2    S+   17:47   0:00 grep --color=auto ruby

  $ gdb
  GNU gdb (GDB) Fedora 8.0.1-36.fc26
  Copyright (C) 2017 Free Software Foundation, Inc.
  # [...]
  (gdb) 
  ```
Once gdb has started up, you should be sitting at an interactive console. Type `attach [PID]` to attach the debugger to the frozen Ruby application.
  ```bash
  (gdb) attach 18198
  Attaching to process 18198
  [New LWP 17919]
  [Thread debugging using libthread_db enabled]
  Using host libthread_db library "/lib64/libthread_db.so.1".
  # [...]
  (gdb) 
  ```

Before we continue, it is worth reviewing what we know about Ruby MRI/YARV/KRI - namely, that it runs on a Virtual Machine written in C. As it runs on a VM, many (if not all?) of the Ruby methods that we know and love are actually defined using C.

Why is this important? When we debug our Ruby application using gdb, any commands that we run are, from my understanding, executed against the __VM__ rather than our Ruby application itself* - as a result, we need to call the equivalent C methods. A (very long) list of available methods can be viewed by typing `call rb_` and pressing tab twice, within gdb.

Now that we've got that out of the way, let's move on to the fun parts!

<em>\* Any corrections here would be greatly appreciated.</em>

### Dumping a backtrace to the application's stdout stream.
In your gdb console, type `call rb_backtrace()` and press enter. If you switch to your Ruby application, you should see a backtrace printed to your application's stdout stream.
```bash
	from my-ruby-script.rb:7:in `<main>'
	from my-ruby-script.rb:3:in `perform_some_cool_function'
	from my-ruby-script.rb:3:in `sleep'
```
Using this backtrace, we can see that the application has "frozen" when running `sleep`, and that sleep was called from within `perform_some_cool_function`.

### Working with multi-threaded applications
You can list the process' threads with `info threads`:
```shell
(gdb) info threads
  Id   Target Id         Frame 
* 1    Thread 0x7f4ed522c080 (LWP 5482) "ruby" 0x00007f4ed4e25918 in pthread_cond_timedwait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  2    Thread 0x7f4ed525c700 (LWP 5570) "ruby-timer-thr" 0x00007f4ed43885a9 in poll () from /lib64/libc.so.6
```

The currently attached thread is denoted by an asterisk (*) next to the thread ID. You can switch threads using `thread [ID]`:
```bash
(gdb) thread 2
[Switching to thread 2 (Thread 0x7f4ed525c700 (LWP 5570))]
#0  0x00007f4ed43885a9 in poll () from /lib64/libc.so.6
```

### Resuming a debugged application
When you attach gdb to a process, it causes the process to "pause". If you would like to resume the process' execution, you will need to `detach` gdb from the process.

To illustrate this, consider the following Ruby code that indefinitely prints a numerical sequence.
```ruby
#!/usr/bin/env ruby
i = 0
until false do
 puts i
 i += 1
end
```

If I run this application, as expected, my console window will begin to flood with numbers, each on a new line.
```bash
# [...]
2116301
2116302
2116303
2116304
```
When I attach gdb to this process and switch back to the process' console window, the application will no longer be printing to the standard output stream (`stdout`).
```shell
(gdb) attach 30688
Attaching to process 30688
[New LWP 30762]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
0x00007f3702ab49a7 in write () from /lib64/libpthread.so.0
Missing separate debuginfos, use: dnf debuginfo-install glibc-2.27-19.fc28.x86_64 libxcrypt-4.0.1-3.fc28.x86_64
```

To resume execution, simply type `detach` and press enter.
```shell
(gdb) detach
Detaching from program: /home/andrew/.rbenv/versions/2.3.0/bin/ruby, process 30688
```

When we switch back to the process' console window, we can see that the numbers have incremented.
```bash
# [...]
2799910
2799911
2799912
2799913
2799914
2799915
```
To reattach gdb to the process, you can simply run `attach [PID]`, as usual.