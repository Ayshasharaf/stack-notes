# Java Regex Notes

A regex (regular expression) is a single string used to represent another string or a set of particular strings.

---

## Syntax Cheat Sheet
 
### 1. Character Classes (match types of characters)
 
| Syntax | Description | Example Matches |
| --- | --- | --- |
| `.` | any single character except a newline (`\n`) | `c.t` matches "cat", "cut", "c*t" |
| `\d` | any digit (`0–9`) | `\d{2}` matches "42" |
| `\D` | any non-digit character | `\D` matches "A", "$" |
| `\w` | any word character (letters, digits, underscore) | `\w+` matches "user_name1" |
| `\W` | any non-word character (spaces, punctuation, symbols) | `\W` matches "!", " " |
| `\s` | any whitespace character (spaces, tabs, newlines) | `\s` matches a tab or space |
| `\S` | any non-whitespace character | `\S` matches visible text characters |
 
### 2. Anchors & Boundaries (match positions, not characters)
 
| Syntax | Description | Example |
| --- | --- | --- |
| `^` | start of a string (or line, in multiline mode) | `^Hello` matches strings beginning with "Hello" |
| `$` | end of a string (or line) | `[0-9]$` matches strings ending with a digit |
| `\b` | word boundary (between a `\w` and a `\W` char) | `\bcat\b` matches "cat" but not "caterpillar" |
| `\B` | non-word boundary | `\Bcat\B` matches "location" (inside "cat") |
 
### 3. Quantifiers & Repetition
 
| Syntax | Description | Example |
| --- | --- | --- |
| `*` | 0 or more times | `a*` matches "", "a", "aaa" |
| `+` | 1 or more times | `\d+` matches "5", "1234" |
| `?` | 0 or 1 time (optional) | `colou?r` matches "color" or "colour" |
| `{n}` | exactly n times | `\d{3}` matches exactly 3 digits |
| `{n,}` | n or more times | `\w{5,}` matches words with 5+ characters |
| `{n,m}` | between n and m times | `[a-z]{2,4}` matches 2 to 4 lowercase letters |
| `*?`, `+?` | lazy modifier — appending `?` to a quantifier makes it match as *few* characters as possible instead of as many | `".*?"` matches text inside quotes minimally |
 
Quantifier examples:
```
a?b   0 or 1 occurrence of preceding element → ab or b
a*b   0 or more occurrence of preceding element → b, ab, aab
a+b   1 or more occurrence of preceding element → ab, aab
```
 
### 4. Character Sets & Ranges
 
| Syntax | Description | Example |
| --- | --- | --- |
| `[abc]` | any single character listed inside the brackets | `[aeiou]` matches any lowercase vowel |
| `[^abc]` | negated set — anything **except** the listed characters | `[^0-9]` matches any non-digit character |
| `[a-z]` | any character within the specified range | `[A-Z]` matches any uppercase English letter |
 
### 5. Groups & Alternation
 
| Syntax | Description | Example |
| --- | --- | --- |
| `(...)` | capturing group — groups items and captures the matched text into a numbered index | `(\d{3})-(\d{2})` |
| `(?:...)` | non-capturing group — groups items without saving the match into memory | `(?:un)?do` |
| `\|` | alternation — acts as "OR" | `li(k\|f)e` matches "like" or "life" |
| `\1`, `\2` | backreference — refers back to text matched by a previous capturing group | `(\b\w+\b)\s+\1` matches duplicate adjacent words |
 
### 6. Lookaround Assertions
 
Lookarounds check text ahead or behind the current position **without** consuming characters into the match.
 
| Syntax | Description | Example |
| --- | --- | --- |
| `(?=...)` | positive lookahead — pattern must follow immediately | `\d(?=px)` matches digits only if followed by "px" |
| `(?!...)` | negative lookahead — pattern must *not* follow immediately | `\d(?!px)` matches digits not followed by "px" |
| `(?<=...)` | positive lookbehind — pattern must precede immediately | `(?<=\$)\d+` matches numbers preceded by a dollar sign |
| `(?<!...)` | negative lookbehind — pattern must *not* precede immediately | `(?<!\$)\d+` matches numbers not preceded by a dollar sign |
 
### 7. Escaping Special Characters
 
To treat a reserved regex metacharacter (`. * ? ^ $ ( ) [ ] { } \ | /`) as a literal character, escape it with a backslash.
```
\.  \*  \\   escaped special characters
\t  \n  \r   tab, linefeed, carriage return
```
Example: `\.` matches a literal dot instead of acting as a wildcard.
 
---


## The 3 things (mental model)


| Piece     | Role                             |
| --------- | -------------------------------- |
| `Pattern` | the rule                         |
| `Matcher` | the worker that applies the rule |
| String    | what gets searched               |


```java
Pattern p = Pattern.compile("\\d+");
```

This builds a rule for "one or more digits." `p` just holds the rule , nothing is searched yet.

```java
Matcher m = p.matcher("order 42, item 7");
```

This says: apply that rule to this string. `m` is now ready to search.

```java
while (m.find()) {
    System.out.println(m.group());
}
```

- `m.find()` : searches from where it last stopped; if it finds a match, returns `true` and stores it.
- `m.group()` : returns what was just found (doesn't search on its own).



### Walkthrough on `"order 42, item 7"` with `\d+`


| Call         | What happens                           | m.group()  |
| ------------ | -------------------------------------- | ---------- |
| 1st `find()` | scans from position 0, finds "42"      | `"42"`     |
| 2nd `find()` | scans from right after "42", finds "7" | `"7"`      |
| 3rd `find()` | scans to end, finds nothing            | loop exits |




### group() before find() → crash

```java
Matcher m = p.matcher("order 42");
m.group();     // CRASH , no find() called yet
m.find();      // now it searches and stores "42"
m.group();     // works , returns "42"
```

`group()` only reads the *last successful match* , it never searches on its own.

### What group(1), group(2) mean

When the pattern has `(parentheses)`, each pair is a numbered capture. `group(0)` (or plain `group()`) is always the whole match. `group(1)` is the first `(...)`, `group(2)` the second, etc.

```java
Pattern p = Pattern.compile("(\\w+)@(\\w+)\\.com");
Matcher m = p.matcher("stitch@example.com");
m.matches();          // run the match first , returns true/false
m.group(0);           // "stitch@example.com"  → whole match
m.group(1);           // "stitch"              → 1st parens
m.group(2);           // "example"             → 2nd parens
```

Think of `(...)` as "and also remember this specific piece separately."

### matches() vs find()

- `m.matches()` : tries to fit the pattern against the **entire** string, start to end, once. Returns `true`/`false`; if `true`, stores that whole-string match so `group()` works after.
- `m.find()` : tries to fit the pattern **anywhere** inside the string, can be called repeatedly to walk forward through multiple matches.

---



## Worked example

```java
String log = "TXN acc-1042 sent to acc-2087, fee acc-9001";

Pattern p = Pattern.compile("acc-\\d{4}");
Matcher m = p.matcher(log);

while (m.find()) {
    System.out.println(m.group());
}
```

Output:

```
acc-1042
acc-2087
acc-9001
```

---



## JS equivalent (for comparison)

```js
const regex = /meow/g;
const text = "meow is crazy meow meow";
console.log(text.replace(regex, "MEEW"));
```

Output:

```
MEEW is crazy MEEW MEEW
```

---
 
## HackerRank Tag Content Extractor
 
Basic regex symbols used here:
 
| Regex | Meaning |
|---|---|
| `<` | match literal `<` |
| `>` | match literal `>` |
| `.` | any character |
| `+` | one or more of the previous thing |
| `[...]` | character set |
| `[^...]` | NOT these characters |
| `(...)` | capturing group |
| `\1` | match the same text captured by group 1 |
 
The regex:
```
<([A-Za-z0-9 ]+)>([^<>]+)</\1>
```
Think of it as: `<tag>content</tag>`
 
Breakdown:
- `<` → opening `<`
- `([A-Za-z0-9 ]+)` → **group 1**: tag name
- `>` → closing `>`
- `([^<>]+)` → **group 2**: content (cannot contain `<` or `>`)
- `</` → closing tag starts
- `\1` → same tag name as group 1
- `>` → closing `>`
### Capturing groups, in action
 
For `<h1>Hello World</h1>`:
```
m.group(1)  → "h1"
m.group(2)  → "Hello World"
m.group()   → "<h1>Hello World</h1>"   — the entire match
```
 
### Backreference `\1`
 
In Java, `\\1` in a string literal represents the regex `\1`. It means: *"match exactly what group 1 matched."*
```
<h1>Hello</h1>   ✅  — closing tag is h1, matches group 1
<h1>Hello</h2>   ❌  — closing tag is h2, group 1 was h1, no match
```
 
### Why `[^<>]+`?
 
`[^<>]+` means: one or more characters that are NOT `<` or `>`. This prevents nested tags from being treated as valid outer content.
```
<h1>Hello</h1>                 → "Hello" ✅
<h1><a>Hello</a></h1>          → outer content invalid; inner "Hello" is valid
```
 
### Java escaping reminder
 
Regex `\1` becomes the Java string `"\\1"`, because Java uses `\` as its own escape character first.
 
### Main lesson
 
Capturing group + backreference = a way to check that two parts of a string are identical:
- `(...)` → capture something
- `\1` → require the same thing again
This pattern (matching opening/closing tags, or detecting duplicated text) is the core use case for backreferences.
 
---
 
## Quick Review: Java Regex Essentials
 
**1. Core mental model (Pattern, Matcher, String)**
- `Pattern p = Pattern.compile("regex");` — compiles the rule into memory
- `Matcher m = p.matcher(text);` — applies that rule to a specific string
- `m.find()` / `m.matches()` — search for the pattern (`find` scans anywhere; `matches` checks the entire string start to finish)
**2. Character classes & quantifiers**
- `[a-zA-Z]` — a character set matching any single letter
- `\w` — any word character (letters, digits, underscore)
- `{7,29}` — a quantifier specifying a range (between 7 and 29 repetitions). Always use commas inside curly braces, not hyphens.
- `[^>]` — a negated set meaning "any character except `>`"
**3. Anchors**
- `^` — asserts the start of a string
- `$` — asserts the end of a string
**4. Groups & backreferences**
- `(...)` (capturing groups) — captures matched text into numbered slots
  - `m.group(0)` = the whole match
  - `m.group(1)`, `m.group(2)` = specific parenthesized pieces
- `\1` (backreference) — reuses text captured by the first group. Essential for validating matching opening/closing HTML/XML tags (e.g. ensuring `<(.+)>` matches its corresponding `</\1>`)
---


## References

- [https://youtu.be/HvryGBPmNl8?si=2pYn9nT0QTSBBUcu](https://youtu.be/HvryGBPmNl8?si=2pYn9nT0QTSBBUcu)
- [https://docs.icewarp.com/Content/IceWarp-Server/Administration-Nodes/Domains%20&%20Accounts/Simple%20RegEx%20Tutorial.htm](https://docs.icewarp.com/Content/IceWarp-Server/Administration-Nodes/Domains%20&%20Accounts/Simple%20RegEx%20Tutorial.htm)
- [https://regexr.com](https://regexr.com)

