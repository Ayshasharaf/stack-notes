# Java String & Character Cheat Sheet for LeetCode DSA

> Quick reference for string manipulation, type conversions, case handling, and space operations commonly used in DSA problems.

---

## 1. Remove / Handle Spaces

```java
// Remove ALL spaces
String s = "hello world";
s.replace(" ", "");              // "helloworld"
s.replaceAll("\\s+", "");        // removes ALL whitespace (spaces, tabs, newlines)

// Remove leading and trailing spaces only
s.strip();                       // Java 11+ (Unicode-aware)
s.trim();                        // older, ASCII-only

// Replace multiple spaces with single space
s.replaceAll("\\s+", " ").strip();   // "hello   world" → "hello world"

s = s.toLowerCase().replaceAll("[^a-z0-9]", "");


// Split by spaces (useful for word problems)
String[] words = s.trim().split("\\s+");   // handles multiple spaces safely
```

**LeetCode use case:** Reverse Words in a String (#151), Valid Palindrome (#125)

---

## 2. String ↔ int / long

```java
// String to int
int n = Integer.parseInt("42");          // throws NumberFormatException if invalid
int n = Integer.parseInt("-10");         // works with negatives

// int to String
String s = Integer.toString(42);
String s = String.valueOf(42);
String s = "" + 42;                      // quick but less readable

// String to long
long l = Long.parseLong("123456789000");

// long to String
String s = Long.toString(l);
String s = String.valueOf(l);

// int to String in a specific base (radix)
String binary  = Integer.toBinaryString(10);   // "1010"
String hex     = Integer.toHexString(255);      // "ff"
String octal   = Integer.toOctalString(8);      // "10"
String base7   = Integer.toString(42, 7);       // any base 2–36

// String (binary/hex) to int
int fromBin = Integer.parseInt("1010", 2);   // 10
int fromHex = Integer.parseInt("ff", 16);    // 255
```

**LeetCode use case:** Add Binary (#67), Number to String conversions, Palindrome Number (#9)

---

## 3. String ↔ char[]

```java
// String to char array
char[] chars = "hello".toCharArray();

// char array to String
String s = new String(chars);
String s = String.valueOf(chars);

// Access individual char
char c = s.charAt(2);     // 'l'

// Iterate over chars
for (char c : s.toCharArray()) { ... }
for (int i = 0; i < s.length(); i++) {
    char c = s.charAt(i);
}
```

**LeetCode use case:** Almost every string problem — sorting, anagram checks, palindromes.

---

## 4. Uppercase / Lowercase

```java
String s = "Hello World";

// Full string
s.toUpperCase();   // "HELLO WORLD"
s.toLowerCase();   // "hello world"

// Single char
char c = 'a';
Character.toUpperCase(c);   // 'A'
Character.toLowerCase(c);   // 'a'

// Check case
Character.isUpperCase(c);   // false
Character.isLowerCase(c);   // true

// Manual bit trick (lowercase → uppercase, works for a-z only)
char upper = (char)(c & ~32);    // 'a' → 'A'
char lower = (char)(c | 32);     // 'A' → 'a'
```

**LeetCode use case:** Valid Palindrome, Case-insensitive comparisons, Capitalize Words.

---

## 5. Char ↔ int (ASCII tricks)

```java
// char to int (ASCII value)
int val = 'a';          // 97
int val = (int)'a';     // 97

// Get 0-based index of a letter
int idx = c - 'a';      // 'a'→0, 'b'→1, ..., 'z'→25
int idx = c - 'A';      // 'A'→0, 'B'→1, ..., 'Z'→25
int digit = c - '0';    // '0'→0, '1'→1, ..., '9'→9

// int back to char
char c = (char)('a' + idx);   // 0→'a', 1→'b', ...
char c = (char)(digit + '0'); // int digit back to char
```

**LeetCode use case:** Frequency maps without HashMap, Caesar Cipher, counting letters.

---

## 6. Character Type Checks

```java
Character.isLetter(c);       // a-z, A-Z
Character.isDigit(c);        // 0-9
Character.isLetterOrDigit(c); // alphanumeric
Character.isWhitespace(c);   // space, tab, newline
Character.isAlphabetic(c);   // broader than isLetter (includes Unicode)

// Manual (faster in tight loops)
boolean isLower = c >= 'a' && c <= 'z';
boolean isUpper = c >= 'A' && c <= 'Z';
boolean isDigit = c >= '0' && c <= '9';
```

**LeetCode use case:** Valid Palindrome (#125) — skip non-alphanumeric chars.

---

## 7. String Building (avoid `+=` in loops!)

```java
// ❌ Slow — creates new String object each time
String result = "";
for (int i = 0; i < n; i++) result += chars[i];

// ✅ Fast — use StringBuilder
StringBuilder sb = new StringBuilder();
sb.append('a');
sb.append("hello");
sb.insert(0, 'x');          // insert at index
sb.deleteCharAt(2);          // delete char at index
sb.reverse();                // in-place reverse
sb.setCharAt(1, 'z');       // modify char at index
sb.charAt(0);                // access
sb.length();                 // length
sb.toString();               // convert back to String
```

**LeetCode use case:** String Compression (#443), Reverse String, building results in sliding window.

---

## 8. String Comparison

```java
// ✅ Always use .equals() for content comparison
s1.equals(s2);
s1.equalsIgnoreCase(s2);    // case-insensitive

// ❌ Never use == for Strings (compares references)
s1 == s2   // unreliable

// Lexicographic comparison
s1.compareTo(s2);            // 0 if equal, <0 if s1<s2, >0 if s1>s2
s1.compareToIgnoreCase(s2);
```

---

## 9. Useful String Methods

```java
s.length()                      // length
s.charAt(i)                     // char at index
s.indexOf('a')                  // first index of char (-1 if not found)
s.indexOf("sub")                // first index of substring
s.lastIndexOf('a')              // last index
s.contains("abc")               // true/false
s.startsWith("he")              // true/false
s.endsWith("lo")                // true/false
s.substring(2)                  // from index 2 to end
s.substring(1, 4)               // index 1 inclusive to 4 exclusive
s.replace('a', 'b')             // replace all char occurrences
s.replace("ab", "xy")          // replace substring
s.split(",")                    // split into array
s.split(",", 2)                 // split with limit
String.join("-", "a","b","c")  // "a-b-c"
String.join("-", list)          // join from List<String>
```

---

## 10. Frequency Map Pattern

```java
// Count character frequencies (only lowercase a-z)
int[] freq = new int[26];
for (char c : s.toCharArray()) {
    freq[c - 'a']++;
}

// Check if two strings are anagrams
Arrays.equals(freq1, freq2);

// With HashMap (for general characters)
Map<Character, Integer> map = new HashMap<>();
for (char c : s.toCharArray()) {
    map.put(c, map.getOrDefault(c, 0) + 1);
}
```

**LeetCode use case:** Valid Anagram (#242), Group Anagrams (#49), Ransom Note (#383).

---

## 11. Palindrome Check Patterns

```java
// Two-pointer approach
boolean isPalin(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        if (s.charAt(l) != s.charAt(r)) return false;
        l++; r--;
    }
    return true;
}

// Using StringBuilder
s.equals(new StringBuilder(s).reverse().toString());

// Valid Palindrome (skip non-alphanumeric, ignore case)
boolean isPalin(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
        while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
        if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r)))
            return false;
        l++; r--;
    }
    return true;
}
```

---

## 12. Quick Reference Table

| Task | Method |
|------|--------|
| Remove all spaces | `s.replaceAll("\\s+", "")` |
| Trim edges | `s.strip()` |
| Split by spaces | `s.trim().split("\\s+")` |
| String → int | `Integer.parseInt(s)` |
| int → String | `String.valueOf(n)` |
| String → char[] | `s.toCharArray()` |
| char[] → String | `new String(chars)` |
| Lowercase | `s.toLowerCase()` / `Character.toLowerCase(c)` |
| Uppercase | `s.toUpperCase()` / `Character.toUpperCase(c)` |
| Char index (a=0) | `c - 'a'` |
| Index to char | `(char)('a' + idx)` |
| Is letter/digit | `Character.isLetterOrDigit(c)` |
| Build string | `StringBuilder` |
| Compare strings | `s1.equals(s2)` |
| Reverse string | `new StringBuilder(s).reverse().toString()` |
| Char frequency | `int[] freq = new int[26]` |
| Binary string | `Integer.toBinaryString(n)` |

---

> **Tip:** In most LeetCode string problems, the go-to pattern is: `toCharArray()` → process with two pointers or frequency array → `new String(chars)` or `StringBuilder.toString()`.