* safety
    * read bash pitfalls, etc.
* subshells

Getting help
------------

### Specific commands

How to get help with a given command depends on whether it's a shell built-in, a
user declared function, an alias, or an executable.

* there are a few commands useful for identifying a command
    * `type` 
    * `type -t`
    * `command -V` will tell you what a command is and display its definition if
    it's a function or alias or its full path if it's a binary.

    ```
    $ command -V command
    command is a shell builtin
    ```

    ```
    $ command -V quote
    quote is a function
    quote () 
    { 
        local quoted=${1//\'/\'\\\'\'};
        printf "'%s'" "$quoted"
    }
    ```

    ```
    $ command -V alert
    alert is aliased to `notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e 's/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//')"'
    ```

    ```
    $ command -V rm
    rm is /usr/bin/rm
    ```

    ```
    $ command -V foo
    bash: command: foo: not found
    ```

* `man` is a program that shows detailed help on various installed Linux
  utilities, but it doesn't directly return info about shell built-ins. Note
  that this can be confusing if there's a name collision. For example, 
  `man read` will provide information about a C function rather than the `read`
  built-in.
* `help` returns information about Bash shell built-ins. 