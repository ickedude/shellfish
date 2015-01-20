# shellfish
shellfish gives the subprocess module a little shell syntax

## Usage

### Command execution

To execute the command, a class must be created for this command. In normal case this task will be taken over by the module at import time or on declaration. Therefor the command must be in your PATH. Than you can create an instance from that class with the appropriate arguments. Now let the module execute the command. You will get the return code, stdout and stderr. If you need the subprocess object to interact with, then call the command instance on your own.
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
```

```py
import shellfish as sh

# creates mount command object and pipe stdout to stdin of column command object
# executes the pipe statement and save stdout into var
pipe = sh.mount() | sh.column('-t')
_, stdout, _ = sh(pipe)

# now a more complex but mindless example
# creates a cat command object and redirect stdout to stdin of the grep command object
# stdin of the pipe statement comes from file /etc/passwd and the result of the pipe statement
# is written to /tmp/nobody.
pipe = (sh.cat('-') | sh.grep(e='nobody')) < '/etc/passwd' > '/tmp/nobody'
ret, _, stderr = sh(pipe)
```
