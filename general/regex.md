# Regex Syntax Runbook

This runbook serves as a comprehensive guide to understanding and utilizing regular expressions (regex). The instructions, patterns, special concepts, and important points provided will assist users in crafting and comprehending regex.

---

## 1. Regex Patterns

| Pattern       | Meaning                                      |
|---------------|----------------------------------------------|
| `\\d`        | Matches any digit (0-9)                      |
| `\\D`        | Matches any character (non-digit)            |
| `\\s`        | Matches any whitespace character (spaces, tabs) |
| `\\S`        | Matches non-whitespace characters             |
| `\\w`        | Matches word characters (digits and letters) |
| `\\W`        | Matches non-word characters                   |
| `\\b`        | Matches a word boundary                       |

| Pattern        | Meaning                                                  |
|----------------|--------------------------------------------------------|
| `a?`          | Matches zero or one occurrence of "a"                  |
| `a*`          | Matches zero or more occurrences of "a"                 |
| `a+`          | Matches one or more occurrences of "a"                  |
| `a{3}`        | Matches exactly three occurrences of "a"                |
| `a{3,}`       | Matches three or more occurrences of "a"                |
| `a{3,6}`      | Matches between three and six occurrences of "a"       |
| `^`           | Matches the start of a line                            |
| `$`           | Matches the end of a line                              |
| `.`           | Matches any single character except newline              |
| `[]`          | Matches one of the characters inside the brackets       |
| `[^]`         | Negates characters within the brackets                  |
| `()`          | Groups characters; can also be used for capturing      |
| `\\n`         | References a captured group using back-reference         |

---

## 2. Special Concepts

### 2.1 Branch Reset Groups
- **Supported By**: PHP, R, C++
- **Syntax**: `(?|(a)|(b)|(c))`
- **Explanation**: Matches either "a", "b", or "c", treating them as the same group for back-references.

### 2.2 Forward Reference
- **Supported By**: JGsoft, .NET, Java, Perl, PCRE, PHP, Delphi, Ruby
- **Syntax**: `(\\2two|(one))`
- **Explanation**: Matches `one` or `onetwo`; `\2` refers back to the first capture group.

### 2.3 Positive Lookahead
- **Supported By**: All
- **Syntax**: `regex1(?=regex2)`
- **Explanation**: Asserts that `regex1` is followed by `regex2` without including `regex2` in the match.

### 2.4 Negative Lookahead
- **Supported By**: All
- **Syntax**: `regex1(?!regex2)`

## Points to Remember 
- When using the dot (.) character, it should be escaped in strings as \\. to ensure the regex is interpreted correctly.