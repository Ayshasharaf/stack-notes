# Java Regex Notes

A regex (regular expression) is a single string used to represent another string or a set of particular strings.

---

## Syntax Cheat Sheet

### Character classes

```
.       any character except newline
\w \d \s   word, digit, whitespace
\W \D \S   not word, digit, whitespace
[abc]      any of a, b, or c
[^abc]     not a, b, or c
[a-g]      character between a & g
```

### Anchors

```
^abc$   start / end of the string
\b \B   word boundary, not-word boundary
```

### Escaped characters

```
\. \* \\   escaped special characters
\t \n \r   tab, linefeed, carriage return
```



### Groups & lookaround

```
(abc)     capture group
\1        backreference to group #1
(?:abc)   non-capturing group
(?=abc)   positive lookahead
(?!abc)   negative lookahead
```



### Quantifiers & alternation

```
a* a+ a?    0 or more, 1 or more, 0 or 1
a{5} a{2,}  exactly five, two or more
a{1,3}      between one & three
a+? a{2,}?  match as few as possible (lazy)
ab|cd       match ab or cd
```



### Quantifier examples

```
a?b   0 or 1 occurrence of preceding element → ab or b
a*b   0 or more occurrence of preceding element → b, ab, aab
a+b   1 or more occurrence of preceding element → ab, aab
```

Alternation example:

```
li(k|f)e   → matches "like" or "life"
```

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



## References

- [https://youtu.be/HvryGBPmNl8?si=2pYn9nT0QTSBBUcu](https://youtu.be/HvryGBPmNl8?si=2pYn9nT0QTSBBUcu)
- [https://docs.icewarp.com/Content/IceWarp-Server/Administration-Nodes/Domains%20&%20Accounts/Simple%20RegEx%20Tutorial.htm](https://docs.icewarp.com/Content/IceWarp-Server/Administration-Nodes/Domains%20&%20Accounts/Simple%20RegEx%20Tutorial.htm)
- [https://regexr.com](https://regexr.com)

