# bash tricks

A collection of bash tricks found in my bash spelunking, ranging from essential
to outright evil.

## Ground rules

These are things I consider to be essential for safe and productive
bash scripting.

### Use a recent version of bash

Bash is a distant descendant of the earliest UNIX shells and, as such, it is
very limited. However, recent versions have introduced numerous features that
can make life easier (or harder).

Preferably use Bash 4.4 at least. Bash 5 is quite recent but worth upgrading to
if you have the option. Everything herein is tested on Bash 4.4.19.

### Strict mode

By default, bash is forgiving to an extreme. Undefined variables and errors are
no problem. bash will just interpolate undefined vars as an empty string and
ignore errors. This is especially useful if you run a command like 
`rm -rf ~/$file_to_deelte` when you meant to write `rm -rf ~/$file_to_delete`.
Your home directory will thank you for keeping it lean and clean.

If, for some reason, you don't agree with this eminently reasonable and normal
behavior, there are some flags to prevent this:

```bash
set -euo pipefail
```

You can also provide longer flag names:

```bash
set -o errexit
set -o noundef
set -o pipefail
```

Both of the above blocks of code are equivalent.

`-e`/`-o errexit` will terminate the script immediately if a command exits with
a nonzero status code, with some mostly-reasonable exceptions (e.g. commands
in an if statement test or used as part of an expression with `&&` or `||`).

`u`/`-o noundef` makes it an error to reference an undefined variable or
parameter.

`-o pipefail` causes pipelines to return a nonzero status if any commands in a
pipeline return nonzero. This fixes a tricky edge case that can slip past 
`errexit`. By default, a pipeline's exit status is always the same as the last
command, even if earlier commands failed. This becomes a problem if we really
wanted those earlier commands to work.

### Top-level code in main(); sourced scripts should never have side effects

It's very common to write shell scripts that execute code at the top level.
This poses two problems: First, any helper code in these scripts is impossible
to reuse directly, since doing so will cause the top-level code to execute.
Second, it becomes easy to scatter the script's main code throughout the file,
potentially intermingling function definitions and code, which obfuscates the
script's behavior and makes it harder to understand.

Instead, a script's main code should live in a function named `main` that is
only called if the script has been called directly. This can be accomplished
with the `BASH_SOURCE` variable.

```bash
is_main() {
    [[ "${BASH_SOURCE[1]}" == "$0" ]]
}
```

### Quote your variables

Here are some important things to understand about bash argument passing:

1. Every bash command has only one argument: An array of strings.
2. Unquoted arguments to bash commands are split on spaces.

Here's what that means: If you pass an unquoted variable to bash and it happens
to contain a space, that variable will be split into multiple arguments, with
likely unwanted side effects.

Suppose we have a directory with files named "foo", "bar", "baz", and
"foo bar baz", and we want to remove the files prefixed with "foo". Here's a
wrong way to do it:

```bash
for f in foo*; do
    echo $f
    rm $f
done
```

Let's see what happens when we try this. (Don't run this in a directory where 
you have any important files.)

```bash
$ touch foo bar baz 'foo bar baz'   # create our files
$ ls                                # yep, they're there
 bar   baz   foo  'foo bar baz'
$ for f in foo*; do                 # naively try to delete them
>   echo $f
>   rm $f
> done
foo
foo bar baz
rm: cannot remove 'foo': No such file or directory
$ ls                                # whoops!
'foo bar baz'
```

Expected outcome: the files named "foo" and "foo bar baz" are removed.

Actual outcome: every file except "foo bar baz" is removed.

## Basic

### Dealing with potentially-undefined variables using parameter expansion

The `-u` flag poses a problem: What if we want to handle an undefined variable? 
With `-u` turned on, undefined variables will kill our script immediately with 
a gross error:

```
my_script.sh: line 4: my_var: unbound variable
```

Old tricks like `[[ "$my_var" == "" ]]` don't work, because bash catches the
undefined variable first. Not to worry, however. bash has some magic to handle
this.

```bash
set -euo pipefail

if [[ "${my_var+set}" == "" ]]; then
    echo "my_var is not set!" >&2
    exit 1
fi
```

Despite `my_var` being undefined, the error message prints out and our 
script works fine. The magic here is [parameter expansion](
https://wiki.bash-hackers.org/syntax/pe), a powerful feature I won't
expain here. (Consult that link and the bash manpage for full details.)
Here are a few common patterns:

#### Optional variable with a default value

The `-` expansion operator will expand an undefined variable to a 
default value.

```bash
echo "Hello ${name-world}!"
# Outputs "Hello world!" if $name is undefined.
```

If you want to provide a default for empty variables as well, `:-` is
the way to go.

```bash
echo "Hello ${name:-world}!"
# Outputs "Hello world!" if $name is undefined or an empty string.
```

#### Check if a variable is defined

##### Using `-`/`:-`

We can easily test if a variable is set using the `-` form with an empty
default:

```bash
if [[ -z "${name:-}" ]]; then
    echo "ERROR: the name variable must be set" >&2
    exit 1
fi
```

However, there's one case where this falls flat: What if we want to allow a 
variable to be blank, but require it to be defined?

##### Using `+`/`:+`

The `+`/`:+` expanders are the inverse of `-`/`:-`. In the expansion
`"${name+set}"`, we'll get an empty string if `name` is undefined and the string
"set" if it is not. Confused? Here's what it looks like at the shell:

```bash
$ unset foo                     # make sure foo isn't set
$ echo "${foo+foo is set}"      # this prints a blank line

$ foo="example value"           # now we set foo to a non-empty value
$ echo "${foo+foo is set}"      # and we get...
foo is set
```

Wait, _huh??_ It seems backwards, but there is a use for this.

```bash
if [[ "${name+set}" != "set" ]]; then
    echo "ERROR: the name variable must be set" >&2
    exit 1
fi
```

Since we used `+` here, the expression `${name+set}` will return the string
"set" if `name` is defined at all, even if it is empty. This allows us to
handle the case where a variable must be defined but is permitted to contain
nothing.

Unfortunately, this also has an edge case:

```bash
$ my_array=()
$ echo "${my_array+is_set}"     # this returns nothing

```

For this, we can use `declare -p`, discussed later.

#### Setting defaults easily

A pattern you're likely to run into is saving default values for reuse. This
is obviously preferable to using the same default value everywhere you
reference a particular variable. Knowing about `-` and `:-`, you may be
inclined to try something like this:

```bash
name="${name:-world}
```

However, this need has already been anticipated. The `=`/`:=` expanders
work like `-`/`:-`, but they also assign the default to the variable as 
appropriate. Bash will try to treat a bare variable assignment as a command if 
you run it at the top level, but we can use the POSIX null command `:` to 
concisely set defaults in one place:

```bash
: "${greeting:=Hello}
: "${name:=world}"
: "${punctuation:=!}"
```

The `:` command is functionally equivalent to `true`. It simply returns 0 and
exits, ignoring its parameters. This means that each expression will be
evaluated, but we don't have to worry about finding a real command to run that
won't do anything undesirable as a side effect.

### Handling undefined variables of all types

We saw above that the `+` parameter expansion variant doesn't work on empty
arrays. Luckily, we have yet another trick up our sleeves.

```bash
$ test=()
# Expanding with + still leads to disappointment.
$ [[ "${test+is_set}" == is_set ]] && echo test is set || echo test is not set
test is not set
# Here's a different idea: What does `declare -p` give us?
$ declare -p test
declare -a test=()
$ declare -p kjlkaj
bash: declare: kjlkaj: not found
$ echo $?
1
$ declare -p test; echo $?      
declare -a test=()
0
# We might be onto something here...
```

`declare` is a bash builtin. It's actually the parent command of `export`,
`local`, `readonly`, and all those. You can do things like declare an array with
`declare -a myvar`. However, `-p` is a special case. Instead of declaring the 
given name, it prints out the `declare` command that would define the variable,
hence this part above:

```bash
$ declare -p test
declare -a test=()
```

We also saw that it returns nonzero if the given name isn't defined. This is the
key. We can test if a variable is defined using `declare -p`, even if it's an
empty array!

Because `declare -p` is a bit noisy, we'll probably want to call it with its
stdout and stderr redirected to /dev/null. This is a bit of typing, so let's
wrap it in a function:

```bash
var_defined() {
    declare -p "$1" >/dev/null 2>/dev/null
}
```

### Handling nonzero exits

Setting `-e` seems to put us in a bit of a bind if we ever expect a
command to return nonzero. Indeed, many commands like `which`, `grep`, 
etc. return nonzero to indicate that no matches were found. Luckily, this
is pretty much free with conditionals. Patterns like this are out:

```bash
do_the_thing | grep -q ERROR
if [[ $? -eq 0 ]]; then
    handle_error
fi
```

Instead, do this:

```bash
if do_the_thing | grep -q ERROR; then
    handle_error
fi
```

### Using `:` and `true` as placeholders for empty functions

`:` was mentioned above as a trick for using the assignment parameter
expansion. It also has another use: filling the body of an as-yet 
undefined function.

Empty function bodies are a syntax error in bash.

```bash
foo() {

}
```

This results in the error: ``my_script.sh: line 6: syntax error near 
unexpected token `}'``

Both `:` and `true` are functions which do nothing besides return zero
and exit, perfect for making a temporary function body until we want to
fill it in:

```bash
foo() {
    true
}

bar() {
    :
}
```

### Formatting output with colors and more

Terminals offer special character codes that can be used to format output. This
is how textual user interfaces like Vim and any ncurses based application work.
Building a full terminal graphics application is quite complex, but it's very
easy to accomplish simple formatting, such as printing colored text, useful for
highlighting error messages and other important output.

You can output raw terminal codes using escapes, for example:

```bash
echo -e '\033[31mThis text is red.\033[0m'
```

However, this is gross. Instead, use tput. tput allows you to generate ANSI 
terminal color codes. This command outputs "This text is red." in red on any 
standard terminal:

```bash
echo "$(tput setaf 1)This text is red.$(tput sgr0)"
```

You can define variables to make things even cleaner:

```bash
RED="$(tput setaf 1)"
GREEN="$(tput setaf 2)"
RESET="$(tput setaf 9)"

echo "${RED}Red text. ${GREEN}Green text. ${RESET}Normal text."
```

More in-depth coverage can be found [here](
    https://wiki-dev.bash-hackers.org/scripting/terminalcodes).

### Namespacing

`:` is a valid character in function names. Google's Shell Style Guide
encourages using double colons `::` to namespace library functions and I mostly
agree.

## Intermediate

### Trapping errors

TODO

### Relative sourcing

By default, the `source` command pulls in code from the working directory. This
is obnoxious if you have a script you want to invoke on the path or from a
standard location.

If you have an executable script that needs to source another file in its
location, this is an easy way of doing so:

```bash
source "$(dirname "$0")/common.sh"
```

***However,*** there's a problem with this. If your executable is symlinked,
`$(dirname "$0")` will resolve the path to the symlink, not the symlink target.
TODO

### Relative sourcing with single symlink support

TODO

### Relative sourcing with unlimited symlink support

TODO

### References

TODO

* parameter expansion
* `declare -n`

## Advanced

### Inspecting the stack

`BASH_SOURCE` has some friends: `FUNCNAME` and `BASH_LINENO`.

From the bash manpage:

> Each element of FUNCNAME has corresponding elements in BASH_LINENO and 
BASH_SOURCE to describe the call stack.  For instance, ${FUNCNAME[$i]} was 
called from the file ${BASH_SOURCE[$i+1]} at line number ${BASH_LINENO[$i]}.

## Outright evil

These are tricks that can be abused for fun, but should be avoided at all costs
in real code unless you _really_ want to frustrate people.

### Overloading variable and function names

Bash has no problem letting you define a function and a variable with the same
name:

```bash
$ foo="hello world"
$ foo() {
>   echo "foo() called"
> }
$ echo "$foo"
hello world
$ foo
foo() called
```

# TODO

- [ ] traps
- [ ] mktemp
- [ ] declaring functions dynamically with `eval` (requires `declare -g`)