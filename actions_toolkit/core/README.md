# `actions.core`

> Core functions for setting results, logging, registering secrets and exporting variables across actions.

## Usage

### Import the package

```python
from actions_toolkit import core
```

#### Inputs/Outputs

Action inputs can be read with `get_input`. Outputs can be set with `set_output` which makes them available to be mapped into inputs of other actions to ensure they are decoupled.

```python
from actions_toolkit import core

my_input = core.get_input('input_name', required=True)
core.set_output('output_key', 'output_val')
```

#### Exporting variables

Since each step runs in a separate process, you can use `export_variable` to add it to this step and future steps environment blocks.

```python
from actions_toolkit import core

core.export_variable('env_var', 'val')
```

#### Setting a secret

Setting a secret registers the secret with the runner to ensure it is masked in logs.

```python
from actions_toolkit import core

core.set_secret('my_password')
```

#### PATH Manipulation

To make a tool's path available in the path for the remainder of the job (without altering the machine or containers state), use `add_path`. The runner will prepend the path given to the jobs PATH.

```python
from actions_toolkit import core

core.add_path('/path/to/my_tool')
```

#### Exit codes

You should use this library to set the failing exit code for your action. If status is not set and the script runs to completion, that will lead to a success.

```python
from actions_toolkit import core

try:
    # Do stuff
    pass
except Exception as e:
    core.set_failed(f'Action failed with error {e}')
```

Note that `set_neutral` is not yet implemented in actions V2 but equivalent functionality is being planned.

#### Logging

Finally, this library provides some utilities for logging. Note that debug logging is hidden from the logs by default. This behavior can be toggled by enabling the [Step Debug Logs](https://github.com/actions/toolkit/blob/main/docs/action-debugging.md#step-debug-logs).

```python
from actions_toolkit import core

my_input = core.get_input('input')
try:
    core.debug('Inside try block')

    if not my_input:
        core.warning('my_input was not set')
    
    if core.is_debug():
        # curl -v https://github.com
        pass
    else:
        # curl https://github.com
        pass
    
    # Do stuff
    core.info('Output to the actions build log')
except Exception as e:
    core.error(f'Error {e}, action may still succeed though')
```

This library can also wrap chunks of output in foldable groups.

```python
from actions_toolkit import core

# Manually wrap output
core.start_group('Do some function')
# do_some_function()
core.end_group()

# Wrap an asynchronous function call
# TODO
```

#### Styling output

Colored output is supported in the Action logs via standard [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code). 3/4 bit, 8 bit and 24 bit colors are all supported.

Foreground colors:

```python
from actions_toolkit import core

# 3/4 bit
core.info('\u001b[35mThis foreground will be magenta')

# 8 bit
core.info('\u001b[38;5;6mThis foreground will be cyan')

# 24 bit
core.info('\u001b[38;2;255;0;0mThis foreground will be bright red')
```

Background colors:

```python
from actions_toolkit import core

# 3/4 bit
core.info('\u001b[43mThis background will be yellow');

# 8 bit
core.info('\u001b[48;5;6mThis background will be cyan')

# 24 bit
core.info('\u001b[48;2;255;0;0mThis background will be bright red')
```

Special styles:

```python
from actions_toolkit import core

core.info('\u001b[1mBold text')
core.info('\u001b[3mItalic text')
core.info('\u001b[4mUnderlined text')
```

ANSI escape codes can be combined with one another:

```python
from actions_toolkit import core

core.info('\u001b[31;46mRed foreground with a cyan background and \u001b[1mbold text at the end');
```

> Note: Escape codes reset at the start of each line

```python
from actions_toolkit import core

core.info('\u001b[35mThis foreground will be magenta')
core.info('This foreground will reset to the default')
```

Manually typing escape codes can be a little difficult, but you can use third party modules such as colored.

```python
from actions_toolkit import core
from colored import fg, attr

core.info(f'{fg("#abcdef")} Hello world!{attr("reset")}')
```

#### Action state

You can use this library to save state and get state for sharing information between a given wrapper action:

In action's `main.py`:

```python
from actions_toolkit import core

core.save_state('pid_to_kill', 12345)
```

In action's `cleanup.py`:

```python
import os, signal

from actions_toolkit import core

pid = core.get_state('pid_to_kill')
os.kill(int(pid), signal.SIGTERM)
```