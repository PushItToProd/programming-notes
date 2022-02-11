* safety
    * read bash pitfalls, etc.
* subshells

Getting help
------------

### Specific commands

How to get help with a given command depends on whether it's a shell built-in, a
user declared function, an alias, or an executable.

* there are a few commands useful for identifying a command
    * `type` (or `command -V`) will tell you what a command is and display its
      definition if it's a function or alias or its full path if it's a binary.

      ```
      $ type type
      type is a shell builtin
      ```

      ```
      $ type quote
      quote is a function
      quote ()
      {
          local quoted=${1//\'/\'\\\'\'};
          printf "'%s'" "$quoted"
      }
      ```

      ```
      $ type alert
      alert is aliased to `notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e 's/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//')"'
      ```

      ```
      $ type rm
      rm is /usr/bin/rm
      ```

      ```
      $ type if
      if is a shell keyword
      ```

      ```
      $ type foo
      bash: type: foo: not found
      ```
    * `type -t` will display just the type of the command

      ```
      $ type -t type
      builtin
      $ type -t quote
      function
      $ type -t alert
      alias
      $ type -t rm
      file
      $ type -t if
      keyword
      $ type -t foo
      ```

* `man` is a program that shows detailed help on various installed Linux
  utilities, but it doesn't directly return info about shell built-ins. Note
  that this can be confusing if there's a name collision. For example,
  `man read` will provide information about a C function rather than the `read`
  built-in.
* `help` returns information about Bash shell built-ins.

