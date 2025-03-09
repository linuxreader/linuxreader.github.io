## Metacharacters and Wildcard
- Metacharacters
	- Special characters that possess special meaning to the shell.
	- Used in pattern matching (a.k.a. filename expansion or file globbing) and regular expressions.
**dollar sign ($)** 
- mark the end of a line
- Used in regular expressions

**caret (^)**
- mark the beginning of a line
- Used in regular expressions

**period (.)**
- Match a single position
- Used in regular expressions

**asterisk (\*)**
- used in pattern matching
- wildcard character
- Matches zero to an unlimited number of characters except for the leading period (.) in a hidden filename.
- Used in regular expressions

**question mark (?)**
- Used in pattern matching
- Wildcard character
- Matches exactly one character except for the leading period in a hidden filename.
- Used in regular expressions	

**pipe (|)**
- Send the output of one command as input to the next.
- Also used to define alternations in regular expressions.
- Can use as many times in a command as you need. (pipeline)	- Can be used as an OR operator (alternation)
	- this|that|other

**angle brackets (< >)**
- Redirections

**curly brackets ({})**
- Used in regular expressions
- Match an element a specific number of times

**square brackets ([])**
- Used in pattern matching
- Wildcard character
- Match either a set of characters or a range of characters for a single character position.
- Order in which they are listed has no importance.
- Range of characters must be specified in a proper sequence such as [a-z] or [0-9].
- Used in regular expressions

**parentheses (())**
- Create a sub shell

**plus (+)**
- Match a character one or more time

**exclamation mark (!)**
- inverse matches

**semicolon (;)**
- Run a second command after the ;

### Quoting Mechanisms
- Disable special meaning of metacharacters

**Backslash (\\)**
- Escape character
- Cancel out a special character's meaning.

**single quotation (‘’)**
- Mask the meaning of all encapsulated special characters.

**double quotation (“”)**
- Mask the meaning of all but the backslash (\), dollar sign ($), and single quotes (‘’).

### Regular expressions (regexp or regex)
- Text pattern or an expression that is matched against a string of characters in a file or supplied input in a search operation.
- Pattern may include a single character, multiple random characters, a range of characters, word, phrase, or an entire sentence.
- Any pattern containing one or more white spaces must be surrounded by quotation marks.

### `grep` command
- Searches the contents of one or more text files or input supplied for a match.

Flags
-i
- case insensitive search

-n
- number the lines

-v
- exclude these lines

-w
- find an exact match for a word

-E
- match one or the other

-e 
- -Use patterns for matching