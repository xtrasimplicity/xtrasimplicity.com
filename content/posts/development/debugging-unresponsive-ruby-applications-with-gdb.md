---
title: "Debugging unresponsive Ruby Applications With gdb"
date: 2018-06-22T07:56:47+10:00
categories: ['Linux', 'Ruby']
tags: ['ruby', 'gdb', 'debugging']
draft: false
---

Every once in a while, I encounter random freezing/hanging when developing Ruby applications and often find myself having to Google to find the correct gdb commands to use to debug these sorts of issues. To make life easier for myself (and hopefully for others out there), I've decided to document them here for future reference. I will (hopefully!) add to this page as I come across new strategies for debugging these sorts of issues.

Firstly, this guide assumes that you have access to a Unix machine with the GNU debugger ([gdb](https://www.gnu.org/s/gdb/)) installed and that you're running plain-old Ruby (MRI/YARV/KRI) - as the methods described on this page rely on the underlying C methods that the Ruby VM calls.

 If you're following along and don't have a frozen Ruby process, you can simulate a frozen/hung process by running a Ruby script that sleeps, such as:

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

Why is this important? When we debug our Ruby application using gdb, any commands that we run are executed against the __VM__ rather than our Ruby application itself - as a result, we need to call the equivalent C methods. A (very long) list of available methods can be viewed by typing `call rb_` and pressing tab twice, within gdb.

Now that we've got that out of the way, let's move on to the fun parts!

### Dumping a backtrace to the application's stdout stream.
In your gdb console, type `call rb_backtrace()` and press enter. If you switch to your Ruby application, you should see a backtrace printed to your application's stdout stream.
```shell
	from my-ruby-script.rb:7:in `<main>'
	from my-ruby-script.rb:3:in `perform_some_cool_function'
	from my-ruby-script.rb:3:in `sleep'
```
Using this backtrace, we can see that the application has "frozen" when running `sleep`, and that sleep was called from within `perform_some_cool_function`.
