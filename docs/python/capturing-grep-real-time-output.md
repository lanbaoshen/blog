# Capturing Grep Real-Time Output

Using `subprocess.Popen` to execute persistent commands and parse the standard output in real-time is a common practice, for example:

=== "shell=False"

    ```python
    import subprocess

    
    ping_popen = subprocess.Popen(['ping', 'lanbaoshen.github.io'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    ```

=== "shell=True"

    ```python
    import subprocess
    

    ping_popen = subprocess.Popen('ping lanbaoshen.github.io', shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    ```

Sometimes, when you already know and want to filter the keywords included in the standard output, 
you will concatenate the `grep` command instead of using `str` or `bytes` function via Python:

=== "shell=False"

    ```python hl_lines="5"
    import subprocess
    

    ping_popen = subprocess.Popen(['ping', 'lanbaoshen.github.io'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    grep_popen = subprocess.Popen(['grep', 'time'], stdin=ping_popen.stdout, stdout=subprocess.PIPE)
    ```

=== "shell=True"

    ```python hl_lines="4"
    import subprocess
    

    grep_popen = subprocess.Popen('ping lanbaoshen.github.io | grep time', shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    ```

However, it will only print all the output within period at certain intervals, especially for commands like `ping`. 
The output is printed out approximately every 5 minutes.

It seems like there's a buffer in somewhere quietly affecting the output print by certain interval instead of real-time.

Check command `grep` options with:

<!-- termynal -->

```
$ man grep

--line-buffered
    Force output to be line buffered. By default, 
    output is line buffered when standard output 
    is a terminal and block buffered otherwise.
```

Get it, the `grep` uses block buffering when standard output is not a terminal. 

If you want to achieve real-time effects, you need to specify `--line-buffered` to set line buffering instead of block buffering:

=== "shell=False"

    ```python hl_lines="5"
    import subprocess
    
    
    ping_popen = subprocess.Popen(['ping', 'lanbaoshen.github.io'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    grep_popen = subprocess.Popen(['grep', '--line-buffered', 'time'], stdin=ping_popen.stdout, stdout=subprocess.PIPE)
    ```

=== "shell=True"
    
    ```python hl_lines="4"
    import subprocess
    

    grep_popen = subprocess.Popen('ping lanbaoshen.github.io | grep --line-buffered time', shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    ```

!!! Warning
    Using line buffering can cause a performance penalty. 
