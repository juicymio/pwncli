# pwncli - Do pwn by cool command line 
`pwncli` is a simple cli tool for pwner, which can help you to write and debug your exploit effectively. You can use `pwncli` through command-line or other python-script. 

Prefix subcommand is supported on `pwncli`. For example, there's a subcommand named `debug`, and you can use this command by `pwncli debug xxxxx` or `pwncli de xxxxx` or `pwncli d xxxxx`. `pwncli` can recoginze the prefix characters and call `debug` finally. However, if the prefix characters matches two or more subcommands, an MatchError will be raised.

Furthermore, it's very easy to extend new commands on `pwncli` by adding your own subcommand file named `cmd_yourcmd.py` on directory `pwncli/commands`. `pwncli` detects and loads all subcommands automatically.

`pwncli` depends on [click](https://github.com/pallets/click) and [pwntools](https://github.com/Gallopsled/pwntools). The former is a wonderful command line interface tool, and the latter is a helpful CTF-toolkit.

# Installation
`pwncli` is supported on any posix-like-distribution system. If you want to do pwn on `wsl` distrbution, `Ubuntu-16.04/Ubuntu-18.04/Ubuntu-20.04` is a good choice. And you have to make sure your `wsl` distribution's name hasn't been changed.
Your need to install `click` and `pwntools` first, and then install `pwncli`:
```
git clone https://github.com/RoderickChan/pwncli.git
cd ./pwncli
pip3 install --editable .
```

# Usage
## pwncli
Help documentation of pwncli will be outputted after exec `pwncli -h` or `pwncli --help`:
```
# pwncli -h

Usage: pwncli [OPTIONS] COMMAND [ARGS]...

  pwncli tools for pwner!
  
  For cli:
    pwncli -v subcommand args
  For python script:
    script content:
        from pwncli import *
        cli_script()
    then start from cli: 
        ./yourownscript -v subcommand args

Options:
  -f, --filename TEXT  Elf file path to pwn.
  -g, --use-gdb        Always use gdb to debug.  [default: False]
  -ns, --no-stop       Use the 'stop' function or not. Only for debug-command
                       using python script.  [default: False]
  -v, --verbose        Show more info or not.  [default: False]
  -h, --help           Show this message and exit.

Commands:
  debug   Debug the pwn file locally.
  remote  Pwn remote host.
```

## debug command
`debug` is a subcommand of `pwncli`, exec `pwncli debug -h` to get it's helpful doc:
```
# pwncli debug -h

Usage: pwncli debug [OPTIONS] [FILENAME]

  FILENAME: The ELF filename.


Options:
  --argv TEXT                     Argv for process.
  -v, --verbose                   Show more info or not.  [default: False]
  -t, --tmux                      Use tmux to gdb-debug or not.  [default:
                                  False]
  -w, --wsl                       Use ubuntu.exe to gdb-debug or not.
                                  [default: False]
  -a, --attach-mode [auto|tmux|wsl-b|wsl-u|wsl-o|wsl-wt]
                                  Gdb attach mode, wsl: bash.exe | wsl:
                                  ubuntu1234.exe | wsl: open-wsl.exe | wsl:
                                  wt.exe wsl.exe  [default: auto]
  -qp, --qemu-gdbremote TEXT      Only used for qemu, who opens the gdb
                                  listening port. Only tmux supported.Format:
                                  ip:port or only port for localhost.
  -gb, --gdb-breakpoint TEXT      Set gdb breakpoints while gdb-debug is used,
                                  it should be a hex address or '\$rebase'
                                  addr or a function name. Multiple
                                  breakpoints are supported.  [default: ]
  -gs, --gdb-script TEXT          Set gdb commands like '-ex' or '-x' while
                                  gdb-debug is used, the content will be
                                  passed to gdb and use ';' to split lines.
                                  Besides eval-commands, file path is
                                  supported.
  -h, --help                      Show this message and exit.

```

## remote command
```
# pwncli remote -h

Usage: pwncli remote [OPTIONS] [FILENAME] [TARGET]

  FILENAME: ELF filename.

  TARGET: Target victim.

  For remote target:
      pwncli -v remote ./pwn 127.0.0.2:23333 --set-proxy=default
  Or to Specify the ip and port:
      pwncli -v remote -i 127.0.0.1 -p 23333

Options:
  -v, --verbose                   Show more info or not.  [default: False]
  -sp, --set-proxy [notset|default|primitive]
                                  Use proxy from config data or not. default:
                                  pwntools context proxy; primitive: pure
                                  socks connection proxy.  [default: notset]
  -i, --ip TEXT                   The remote ip addr.
  -p, --port INTEGER              The remote port.
  -h, --help                      Show this message and exit.
```

## config file
A config file will be read if it exists. The path is `~/.pwncli.conf`

`pwncli` reads data from config file if some option values are not gived.

The example `~/.pwncli.conf`:
```
[context]
log_level=notset


[remote]
ip=127.0.0.1

[proxy]
type=http
host=localhost
port=80
username=admin
passwd=admin123
rdns=True
```

## example
### use `debug` command through cli
There is an elf file named `pwnme` and your `exp.py`.

When you are in a tmux window and you want to set a breakpoint at `malloc` and `free`:
```
pwncli -v debug ./pwnme -t -gb malloc -gb free
``` 

or to specify your gdb-script `./script`:
```
pwncli -v debug ./pwnme -t -gs "./script"
```

When you use `wsl` and `open-wsl.exe` to debug pwn file:
```
pwncli -v debug ./pwnme -w -a wsl-o -gs "b malloc;b free;directory /usr/src/glibc/glibc-2.23/malloc"
```

### use `debug` command through python-script
Content of your own script `exp.py`:
```python
#!/usr/bin/python3
from pwncli import *

cli_script()

if gift['debug']:
    elf = gift['elf']
    libc = gift['libc']
elif gift['remote']:
    libc = ELF('./libc-2.23.so')

p:tube = gift['io']
p.sendlineafter('xxx', payload)
stop() # stop to gdb debug
leak_addr = u64(p.revn(8))
log_address('leak_addr', leak_addr) # log address 
p.interactive()
```
and then, use your script on cli:
```
./exp.py debug ./pwnme -t -gb malloc
```

### use `remote` command
Specify ip and port:
```
pwncli remote ./pwnme 127.0.0.1:23333
```

If the default ip has been given in `~/.pwncli.conf`, you can just input the port:
```
pwncli remote -p 23333
```

Use proxy, the proxy data must be written in `~/.pwncli.conf`:
```
pwncli re --set-proxy=default
```
The `default` proxy is to set `context.proxy`, and the `primitive` proxy is to use `remote.fromsocktet` and set proxy using `socks` and `socket` modules.
