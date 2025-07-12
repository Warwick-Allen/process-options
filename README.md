# NAME

`process-options` - prints bash commands to handle option arguments

# SYNOPSIS

    . <(process-options $0)   # (from within a bash script)

# DESCRIPTION

When `process-options` is sourced from a bash script, it:

1. Extracts the comments from the source of the shell script from which it has been
invoked, and stores that into a `help` variable (which is in the scope of the
invoking script).
2. Uses the _Options_ section of the help comments to build an `options`
associative array.  The keys of this array are of the form of
_&lt;short option character>&lt;field name>_, where _&lt;field name>_ is one
of:

    - `argument`

        The argument accepted by this option.  Options can take a maximum of one
        argument.

    - `description`

        An explanation of the option.

    - `long`

        The long name for this option.

    - `short`

        The short (single-character) name for this option.

    - `value`

        The current value of the option.  For switch options, this will be a string
        ("true" or "false") or unset (which is equivalent to "false").

    E.g., `options[xvalue]` references the value for the option `x`.

3. Using `getopt`, strips the option arguments from `$@`, using them to populate
the respective `?value` fields in the `options` associative array.
4. Provides an `option` function for convinient getting and setting of the
`options` array. Its behaviour varies depending on the number of arguments it
receives:
    - `option` (with no arguments)

        Displays the keys of `options`.

    - `option <opt-char>` (e.g., `option x`)

        Displays the current value of `options[<opt-char>value]`.  If `options`
        does not contain the key `<opt-char>value`, "false" is displayed for switch
        options (i.e., those that don't take an argument) and nothing is displayed for
        options that do take an argument.

    - `option <opt-char> <field>` (e.g., `option x description`)

        Displays the current value of `options[<opt-char><field>]`.

    - `option <opt-char> <field> <value>...`

        Sets `options[<opt-char><field>]` to `<value>`.  If there are more
        than three arguments, the final ones are concatenated into `<value>`.

### Expected format of the comment block

This script scans for comment lines that either have a hash symbol ("#")
followed by a space, or a hash symbol as the only character on the line.  The
"help" comment block of the file being scanned starts at the first of these
comment lines, and continues until the first empty line (or line containing only
spaces or space-like characters).  The "options" section start after a line
starting with `# Options:`.  Each option desciption has the form:

    #  -<short name>  --<long name>  <argument>  <description>
    #                                            Default: <default>

The argument and the default line are optional.  The description may span
multiple lines.

### Passing arguments

When the options are sent to the bash script:

- Short options that don't take arguments may be grouped (`-ab` is equivalent to
`-a -b`).
- Long names are always prefixed with two hyphens, and cannot be grouped.
- `--` causes all following arguments to be treated as position parameters, not
as options, even if they start with a hyphen.

### Example script - `process-options-example.sh`

    #!/usr/bin/bash

    # process-options-example.sh - shows how to use process-options
    #
    # Usage:  .  process-options-example.sh
    #
    # This is a bash script that examples the use of process-options.
    #
    # Options:
    #   -a --Aa x    The -a (or --Aa) option takes a parameter "x".
    #                Default: Default value for a
    #   -b --Bb      The -b/--Bb switch does not take any parameters, but it does
    #                have a rather long description.

    . <(process-options $0)

    echo "'a' is set to '$(option a)'"
    if $(option b)
    then echo "'b' is set"
    else echo "'b' is not set"
    fi

    option a value new value
    echo "'a' is now set to '$(option a)'"
    option
    echo "The positional arguments are: $@"

### Output

    > ./process-options-example.sh -a 1 one --Bb two
    'a' is set to '1'
    'b' is set
    'a' is now set to 'new value'
    aargument adescription along ashort avalue bdescription blong bshort bvalue
    The positional arguments are: one two

    > ./process-options-example.sh -- -b two
    'a' is set to 'Default value for a'
    'b' is not set
    'a' is now set to 'new value'
    aargument adescription along ashort avalue bdescription blong bshort
    The positional arguments are: -b two

# OPTIONS

- **--generate-documentation \[type\]**

    Generate a _type_ documentation file for this script, where _type_ is one of:

    - html

        Creates an HTML file, (_process-options.html_).

    - markdown

        Creates a Markdown file (_process-options.md_).

    - all

        Creates both of the above files.

    Not specifying _type_ is the same as specifying "all".

- **-h,--help**

    Print usage information, then exit.

- **--man**

    Print this script's full documentation, then exit.

# CAVEATS

The calling script must be invoked as a bash executable, not called with the
`source` command.

# SEE ALSO

- [getopt(1)](https://man7.org/linux/man-pages/man1/getopt.1.html)

# AUTHORS

- [Warwick Allen](https://github.com/Warwick-Allen)

# COPYRIGHT AND LICENCE

Copyright (c) 2025 Warwick Allen

This is free software; you can redistribute it and/or modify it under the same
terms as the Perl 5 programming language system itself.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
