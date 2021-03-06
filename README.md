# shellfish
shellfish gives the python `subprocess` module a kind of shell syntax

## Usage

### Command Creation and Execution

To execute a command, a class must be created for this command. In normal case this task will be taken over by the module at import time or on declaration. Therefor the command must be in your `PATH`. Than you can create an instance from that class with the appropriate arguments. Now let the module execute the command. You will get the return code, stdout and stderr. If you need the `subprocess.Popen` object to interact with, then call the command instance on your own. You are also able to change the current working directory for each child process by using the 'cd()' method.
```py
# import shellfish module and shorten name
import shellfish as sh
# let shellfish module create classes of the cat and echo command
from shellfish import cat, echo

# create instance from echo class imported by shellfish
echo_cmd = echo('test')
# let shellfish create a touch command class and return a instance of that class
touch_cmd = sh.touch('/tmp/shellfish.test')

# let shellfish execute the command and return return code, stdout and stderr
ret, stdout, stderr = sh(echo_cmd)

# call the touch command instance on your own to get the subprocess object,
# then interacte with the object
subproc = touch_cmd()
stdout, stderr = subproc.communicate()
ret = subproc.wait()

# if you are only interested in the return code
ret = sh.call(sh.touch('/tmp/shellfish.test'))

# now the same statement, but shellfish raises an exception if the return code is not zero
try:
    ret = sh.check_call(sh.touch('/tmp/shellfish.test'))
except sh.CalledProcessError:
    pass

# if you are only interested in the output and error conditions
try:
    stdout = sh.check_output(sh.echo('test') | sh.cat('-', '/tmp/shellfish.not_existing'))
except sh.CalledProcessError as e:
    print(e.returncode)
    print(e.output)

# changing the working directory
ret = sh.call(sh.touch('shellfish.test').cd('/tmp'))
```

### Redirection

You can redirect the following types to stdin: a file handle or file object, a file name as `str` or a `str` or `bytes` as heredoc. Use `>` in front of a command or `<` after the command. Be careful if you change the current working directory and are working with relative filenames in the redirections. Think of kind of subshell: `(cd /tmp; cat 'shellfish.test') > 'shellfish.test2'`. `cat` will search for `shellfish.test` in `/tmp`, but will write `shellfish.test2` to the current working directory of the parent process.
```py
import shellfish as sh

# two different ways to redirect a file, providing the file name as string
cmd = sh.cat('-') < '/tmp/shellfish.test'
cmd = '/tmp/shellfish.test' > sh.cat('-')

# same different ways to redirect a file object
f = open('/tmp/shellfish.test', 'r')
cmd = sh.cat('-') < f
cmd = f > sh.cat('-')

# redirect a string to stdin
cmd = "my heredoc" >= sh.cat('-')
cmd = sh.cat('-') <= "my shellfish test"
# redirect bytes to stdin
cmd = b"my heredoc" >= sh.cat('-')
cmd = sh.cat('-') <= b"my shellfish test"
```

The same applies for stdout and stderr. Use `>` after the command to redirect stdout and `>=` to redirect stderr.
```py
import shellfish as sh

# two different ways to redirect a file, providing the file name as string
cmd = sh.echo('my shellfish test') > '/tmp/shellfish.test'
cmd = sh.cat('/tmp/shellfish.test') >= '/tmp/shellfish2.test'

# same different ways to redirect a file object
f = open('/tmp/shellfish.test', 'r')
cmd = sh.cat('-') < f
cmd = f > sh.cat('-')
```

If you want to redirect stderr to stdout, then use the shellfish constant `STDOUT`.
```py
import shellfish as sh
from shellfish import STDOUT

# redirect stderr to stdout
cmd = sh.cat('/tmp/shellfish.test') >= STDOUT
```

Now comes the tricky part, combining the redirections. If you want to redirect stdin and stdout or stderr, then the command object must be in the middle. If you want to redirect stdout and stderr at the same time, then you have to group stdout and stderr as `list` or `tuple`. That is because the comparison operators have equal precedence and evaluation is done from left to right. That means `cmd < stdin > stdout` is evaluated in python like `cmd < stdin and stdin > stdout`. `stdin > stdout` will be `True` or `False`, but we expect to get the modified command object.
```py
# redirect stdin and stdout
cmd = '/tmp/shellfish.test' > sh.cat('-') > '/tmp/shellfish2.test'

# redirect stdout and stderr
cmd = sh.cat('/tmp/shellfish.test') > ('/tmp/shellfish2.test', '/tmp/shellfish3.test')

# redirect stdin, stdout and stderr
cmd = '/tmp/shellfish.test' > sh.cat('-') > ('/tmp/shellfish2.test', '/tmp/shellfish3.test')
```

### Pipelines

Like in shell a sequence of commands, where the stdout of a command is connected via a pipe to another command, is created with `|`. Also pipelines have stdin, stdout and stderr. If you set stdin of a pipeline, then the first commands stdin of the pipeline is set. If you set stdout, then the last commands stdout of the pipeline is set. If you set stderr, then all commands of the pipeline get the stderr set. Changing the current working directory of pipelines is currently not supported.
```py
import shellfish as sh

# redirect mount stdout to column stdin
pipe = sh.mount() | sh.column('-t')
_, stdout, _ = sh(pipe)

# now a more complex but mindless example
# creates a cat command object and redirect stdout to stdin of the grep command object
# stdin of the pipe statement comes from file /etc/passwd and the result of the pipe statement
# is written to /tmp/nobody.
pipe = '/tmp/shellfish.test' > (sh.cat('-') | sh.grep(e='heredoc') | sh.wc('-l')) >= '/tmp/shellfish2.test'
```
