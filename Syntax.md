# TYGL syntax

This is a simple specification for the TYGL configuration language.

# Comments
Comments are enclosed by parentheses `( )` and can appear anywhere outside of strings.
They are ignored as a semantic part of the file.
Example:
```tygl
Score: 300 (the last achieved score)
```

Parentheses inside a comment must be balanced, and the nested ones are just part of the comment.
Quotes have no meaning inside a comment,
but the escape sequences `\(` and `\)` can be used to write an unbalanced parenthesis.
Example:
```tygl
(Old name: "Smith \) Jones")
```

A comment must end on the same line where it starts, otherwise it is an error.
The only exception is a multiline comment,
which is opened by `(` immediately followed by the end of the line.
It ends with the matching `)` on any of the following lines.
Whitespace between that `(` and the end of the line is an error.
Example:
```tygl
(
This file contains some data...
Edit it as needed!)
```

Every comment, including a multiline one, acts like a single space.
It never ends a line, so a tuple can continue after it.
Example:
```tygl
Scores: 9.8 (
    The second score was removed
    after the review.
) 9.1
```

When a comment is between two values, a space (or a comma) is required before or after it.
A comment glued to a value on just one side is fine,
but a comment that is the only thing separating two values is an error.
Example:
```tygl
Weight: 75kg(approximately) 80kg (allowed)
Duration: 1h(approximately)6min (error)
```

Comments can be used in identifiers too, still acting like a single space.
Combined with trimming and collapsing of spaces, this keeps the identifier tidy.
Example:
```tygl
Weight (in kg): 75 (the identifier is "Weight")
Mass (in kg) of Earth: 5.97219_+24 (the identifier is "Mass of Earth")
```

# Values
There are 4 types of values: dictionaries, lists, numbers and strings.

## Dictionaries
Dictionaries are sets of key-value pairs, called **fields**, enclosed by `{ }`.
Each field is named by an **identifier**
(containing all symbols except `:`, `,`, `( )` and newline,
with the escape sequence `\` supporting any codepoint),
followed by `:` and a *tuple* of **values**.
Example:
```tygl
Today's mentioned trivia: {
    Mass of Earth: 5.97219_+24kg
    Tallest statue: "Statue of Unity"
    Most remote inhabited archipelago: "Tristan da Cunha"
}
```

Fields are separated by newlines or commas `,`, and the two can be mixed freely.
Lines containing only whitespace or comments are ignored, so empty lines are just formatting.
Two commas in a row, or a comma before the first or after the last field of a line are errors.
Both `{}` and `{ }` are empty dictionaries.
Example:
```tygl
Point: {X: 12.5, Y: 7.25}
Size: {
    Width: 800, Height: 600

    Depth: 1
}
Empty: {}
Error: {A: 1, B: 2,}
```

Tuples containing one value are still tuples.
A field can't be left blank: an empty value is written as `-` (see Tuples).

A comma inside an identifier is an error. Use the escape sequence `\,` instead.
Example:
```tygl
Width\, height: 800 600
```

Whitespace around the identifiers is trimmed,
and every run of spaces and tabs inside them is collapsed into a single space.
If you *really* need an identifier to start or end with a space,
or to contain several spaces in a row, use the escape sequence `\_`.

The file itself is also a dictionary, just not enclosed by `{ }`.

If a key appears multiple times, and its value only contains a dictionary,
the two dictionaries are merged.
The same applies for the keys appearing in both dictionaries.
In other cases, multiple appearences of a key is an error.
Example:
```tygl
User: {
    Name: "John"
    Score: 300
    Status: {
        Attempts: 3
    }
}
User: {
    Quote: "I'm the best!"
    Status: {
        Playtime: 5h 47min
    }
}
```
...is the same as:
```tygl
User: {
    Name: "John"
    Score: 300
    Quote: "I'm the best!"
    Status: {
        Attempts: 3
        Playtime: 5h 47min
    }
}
```


## Lists
Lists are series of **items** enclosed by `[ ]`. Each item is a tuple.
Items are separated by newlines or commas `,`, and the two can be mixed freely.
Just like in dictionaries, lines containing only whitespace or comments are ignored,
and two commas in a row, or a comma before the first or after the last item of a line, are errors.
An empty item is written as `-`, and it can also carry a comment.
Both `[]` and `[ ]` are empty lists.
Example:
```tygl
Scores: [
    9.8, 8.5
    9.1
]
Points: [12.5 7.25, 3.0 1.5]
Slots: [
    1, -, 3

    - (reserved for the bonus round)
    5
]
Empty: []
Error: [1, 2,]
```

## Numbers
Numbers can be both integers and decimal (with decimal dot `.` only), signed with an optional `+` or `-`.
The digits before the decimal dot can be left out, but a number always needs at least one digit.
A dot is only a decimal dot when a digit follows it, otherwise it starts the number's suffix (see below).
Example:
```tygl
Ratio: .5
Offset: -.25
Dot: . (a simple string)
Version: 5. (the number 5 with the suffix ".")
```

Just for readability numbers can contain underscores between digits for spacing and grouping digits.
Example:
```tygl
Long number: 1_000_000
```

Scientific notation is used by appending the exponent after the significand.
The exponent is appended after the last underscore `_`
and always has either the `+` or `-` sign.
Example:
```tygl
Number in scientific notation: 6.02_+23
```

Binary, octal and hexadecimal numbers are also supported:
- prefix `0x` starts a hexadecimal number. Letters `a-f` and `A-F` are used for digits.
- prefix `0b` starts a binary number
- prefix `0o` starts an octal number.
  Just `0` as a prefix doesn't start an octal number, unlike common implementations.
A prefix must be followed by at least one digit of its base,
otherwise the whole value is a simple string.
Example:
```tygl
Binary sequence: 0b10_1100_0101
Code: 0xg (a simple string)
```

Numbers with a prefix can have a fractional part too, written in the same base.
Example:
```tygl
One and a half: 0x1.8
```

Exponents in scientific notation are always written in decimal,
but modify the number's significand in it's used base.
```tygl
Big binary: 0b1_+9 (equal to 0b10_0000_0000)
```

Numbers can have suffixes, usually used as units.
The suffix starts with the first symbol that isn't otherwise expected at that part of the number
and  can contain any character except the special ones used in values,
`( ) [ ] { } " ' ,` or newlines and spaces. For those characters you can use the escape sequence `\`.
Example:
```tygl
Weight: 75kg
Distance: 5km
```

## Strings
Strings are textual values. They can contain any sequence of characters.

### Normal strings
Normal strings are enclosed by single or double quotes.
Example:
```tygl
Name: "John Smith"
```

In single-quoted strings `"` character can be used unescaped,
and in double-quoted strings `'` can be used unescaped.
Escape sequence `\` can be used in normal strings for special characters, like newlines.
Example:
```tygl
Line: "John said \"Hello world!\""
```

### Simple strings
Simple strings are still pieces of text, but not enclosed in quotes.
They are often used for programmer-friendly identifiers,
to denote the format of values, or just to simplify writing of one-worded text.
Any unquoted value that isn't a number is a simple string.
It can contain any character except the special ones used in values,
`( ) [ ] { } " ' ,` or newlines and spaces. For those characters you can use the escape sequence `\`.
Example:
```tygl
Blood type: AB+
```

A lone `-` is not a simple string but an empty tuple, so write `\-` for a string containing just a minus.
Simple strings that merely start with a sign, like `-v` or `+x`, are fine.

### Raw strings
Raw strings can be multiline. They are written with the double quote character directly followed by a new line.
Each line of the string needs to start indented up to the opening quote, plus an additional space.
The expected sequence in order:
1. Exactly the same whitespace indentation as the previous line (the string of ` ` and `\t` characters before non-whitespace)
2. A string of space (` `) characters with the length of the number of characters before the opening quote, after the indent in the previous line
3. A single whitespace (` ` or `\t`), or the closing quote to end the string
The closing quote is aligned to the opening quote, without an additional space.

All lines except the ones with opening and closing quotes are separate lines of the string,
separated by a newline character.
Remember that the last line doesn't end with a newline character,
unless of course an empty line is written before the closing quote.
Lines with less whitespace than needed and nothing else are treated as empty lines too.
The lines are interpreted literally,
with quotes and `\` characters just being normal characters of the string.
Example:
```tygl
Comment: "
          Enjoyed the scenic route.
          Planning to bring friends next time.
         "
```

Note: due to some editors making it harder to control indentation
enough that the string lines consistently align with the opening quote on a busy line,
it is recommended to put raw string opening quotes on their own lines.
Example:
```tygl
Comment: \
    "
     Enjoyed the scenic route.
     Planning to bring friends next time.
    "
```

## Tuples
The tuples are lists of values separated by a space ` `. All values are contained in tuples.
It can be of any size, and contain values of any types.
The space separating values is always required,
even before special characters starting some values, like `[ ] { } " '`.
Comments and line continuations count as spaces here, as described in their sections.
Example:
```tygl
Favourite color: rgb 240 98 146
```

A comma `,` ends the tuple, as described for dictionaries and lists.
No space is required around it.
Example:
```tygl
Pairs: [1h,6min, 2h,15min] (4 items)
```

An empty tuple is written as a single minus `-`, which must be the only value in that tuple.
A tuple can never be left blank.
Example:
```tygl
Middle name: -
Missing: (error)
Mixed: 1 - 2 (error)
```

End of the line usually signifies the end of the tuple,
but a backslash `\` directly followed by the end of the line continues the tuple on the next line.
This is a line continuation, not an escape sequence.
It always ends the value before it and acts as the separating space,
so no space is required before the backslash.

A line continuation inside a normal string is an error.
Inside a multiline comment, the ending backslash means nothing and is ignored.
In raw strings it is a normal character, like everything else there.

Example:
```tygl
I'm a multiline tuple: 1 two \
    3.0 0b1_+2 "five"
```

# Escape sequences
Escape sequences are used to write special characters not directly allowed in strings,
identifiers and number suffixes, or those that are hard to type.
The characters that are part of the escape sequence are never interpreted as special TYGL symbols.
They start with a backslash `\` and can be followed by:
- `n` - newline character
- `_` - normal whitespace
- `:` - colon
- `'` - apostrophe or single quote (not interpreted as string begin/end)
- `"` - double quote (not interpreted as string begin/end)
- `(` - opening parenthesis
- `)` - closing parenthesis
- `[` - opening bracket
- `]` - closing bracket
- `{` - opening brace
- `}` - closing brace
- `,` - comma
- `-` - minus (not interpreted as an empty tuple)
- `t` - tab
- `v` - vertical tab
- `r` - carriage return
- `b` - backspace
- `a` - alert (bell, beep), used in some terminals
- `f` - formfeed page break
- `e` - escape character symbol, used in some renderers and terminals
- `0` - Null character. While supported, using it in TYGL is discouraged
- `x` and 2 hex digits - The byte of the specified hexadecimal value
- `u` and 4 hex digits - Unicode code point of the specified hexadecimal value below 0x10000 (65536)
- `U` and 8 hex digits - Unicode code point of the specified hexadecimal value

A backslash at the end of a line is not an escape sequence.
It is a line continuation, described in the Tuples section.

While some of these characters are supported in some places like in identifiers,
they can still be written using escape sequences.

In raw (multiline) strings all characters are interpreted literally until the end of the line.
That means you can't use escape sequences in those strings,
but you can write any character or byte sequence as it is.