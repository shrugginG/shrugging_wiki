[scala](https://www.scala-lang.org/api/2.13.x/scala/index.html).[util](https://www.scala-lang.org/api/2.13.x/scala/util/index.html).[matching](https://www.scala-lang.org/api/2.13.x/scala/util/matching/index.html)

# [Regex](https://www.scala-lang.org/api/2.13.x/scala/util/matching/Regex$.html)

### Companion [object Regex](https://www.scala-lang.org/api/2.13.x/scala/util/matching/Regex$.html)

#### class**Regex** extends [Serializable](https://www.scala-lang.org/api/2.13.x/scala/index.html#Serializable=java.io.Serializable)

A regular expression is used to determine whether a string matches a pattern and, if it does, to extract or transform the parts that match.

##### Usage

This class delegates to the [java.util.regex](https://docs.oracle.com/javase/8/docs/api/java/util/regex/package-summary.html) package of the Java Platform. See the documentation for [java.util.regex.Pattern](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html) for details about the regular expression syntax for pattern strings.

An instance of `Regex` represents a compiled regular expression pattern. Since compilation is expensive, frequently used `Regex`es should be constructed once, outside of loops and perhaps in a companion object.

The canonical way to create a `Regex` is by using the method `r`, provided implicitly for strings:

（规范的创建正则匹配的方法是使用scala为字符串隐式提供的方法`r`）

```scala
val date = raw"(\d{4})-(\d{2})-(\d{2})".r
```

Since escapes are not processed in multi-line string literals, using triple quotes avoids having to escape the backslash character, so that `"\\d"` can be written `"""\d"""`. The same result is achieved with certain interpolators, such as `raw"\d".r` or a custom interpolator `r"\d"` that also compiles the `Regex`.

("\\d"应该被写为“”“\d”“”)

##### Extraction

To extract the capturing groups when a `Regex` is matched, use it as an extractor in a pattern match:

（可以使用模式匹配来提取正则匹配到的内容）

```scala
"2004-01-20" match {
  case date(year, month, day) => s"$year was a good year for PLs."
}
```

To check only whether the `Regex` matches, ignoring any groups, use a sequence wildcard:

(如果只是检测是否匹配成功)

```
"2004-01-20" match {
  case date(_*) => "It's a date!"
}
```

That works because a `Regex` extractor produces a sequence of strings. Extracting only the year from a date could also be expressed with a sequence wildcard:

```scala
"2004-01-20" match {
  case date(year, _*) => s"$year was a good year for PLs."
}
```

In a pattern match, `Regex` normally matches the entire input. However, an unanchored `Regex` finds the pattern anywhere in the input.

(采用unanchored会在整个String中寻找匹配到的内容)

```scala
val embeddedDate = date.unanchored
"Date: 2004-01-20 17:25:18 GMT (10 years, 28 weeks, 5 days, 17 hours and 51 minutes ago)" match {
  case embeddedDate("2004", "01", "20") => "A Scala is born."
}
```

##### Find Matches

To find or replace matches of the pattern, use the various find and replace methods. For each method, there is a version for working with matched strings and another for working with `Match` objects.

For example, pattern matching with an unanchored `Regex`, as in the previous example, can also be accomplished using `findFirstMatchIn`. The `findFirst` methods return an `Option` which is non-empty if a match is found, or `None` for no match:

```scala
val dates = "Important dates in history: 2004-01-20, 1958-09-05, 2010-10-06, 2011-07-15"
val firstDate = date.findFirstIn(dates).getOrElse("No date found.")
val firstYear = for (m <- date.findFirstMatchIn(dates)) yield m.group(1)
```

To find all matches:

```scala
val allYears = for (m <- date.findAllMatchIn(dates)) yield m.group(1)
```

To check whether input is matched by the regex:

(检验是否匹配成功)

```scala
date.matches("2018-03-01")                     // true
date.matches("Today is 2018-03-01")            // false
date.unanchored.matches("Today is 2018-03-01") // true
```

To iterate over the matched strings, use `findAllIn`, which returns a special iterator that can be queried for the `MatchData` of the last match:

```
val mi = date.findAllIn(dates)
while (mi.hasNext) {
  val d = mi.next
  if (mi.group(1).toInt < 1960) println(s"$d: An oldie but goodie.")
}
```

Although the `MatchIterator` returned by `findAllIn` is used like any `Iterator`, with alternating calls to `hasNext` and `next`, `hasNext` has the additional side effect of advancing the underlying matcher to the next unconsumed match. This effect is visible in the `MatchData` representing the "current match".

```
val r = "(ab+c)".r
val s = "xxxabcyyyabbczzz"
r.findAllIn(s).start    // 3
val mi = r.findAllIn(s)
mi.hasNext              // true
mi.start                // 3
mi.next()               // "abc"
mi.start                // 3
mi.hasNext              // true
mi.start                // 9
mi.next()               // "abbc"
```

The example shows that methods on `MatchData` such as `start` will advance to the first match, if necessary. It also shows that `hasNext` will advance to the next unconsumed match, if `next` has already returned the current match.

The current `MatchData` can be captured using the `matchData` method. Alternatively, `findAllMatchIn` returns an `Iterator[Match]`, where there is no interaction between the iterator and `Match` objects it has already produced.

Note that `findAllIn` finds matches that don't overlap. (See [findAllIn](https://www.scala-lang.org/api/2.13.x/scala/util/matching/Regex.html#findAllIn(source:CharSequence):scala.util.matching.Regex.MatchIterator) for more examples.)

```
val num = raw"(\d+)".r
val all = num.findAllIn("123").toList  // List("123"), not List("123", "23", "3")
```

##### Replace Text

Text replacement can be performed unconditionally or as a function of the current match:

```
val redacted    = date.replaceAllIn(dates, "XXXX-XX-XX")
val yearsOnly   = date.replaceAllIn(dates, m => m.group(1))
val months      = (0 to 11).map { i => val c = Calendar.getInstance; c.set(2014, i, 1); f"$c%tb" }
val reformatted = date.replaceAllIn(dates, _ match { case date(y,m,d) => f"${months(m.toInt - 1)} $d, $y" })
```

Pattern matching the `Match` against the `Regex` that created it does not reapply the `Regex`. In the expression for `reformatted`, each `date` match is computed once. But it is possible to apply a `Regex` to a `Match` resulting from a different pattern:

```
val docSpree = """2011(?:-\d{2}){2}""".r
val docView  = date.replaceAllIn(dates, _ match {
  case docSpree() => "Historic doc spree!"
  case _          => "Something else happened"
})
```

- Self Type

  [Regex](https://www.scala-lang.org/api/2.13.x/scala/util/matching/Regex.html)

- Annotations

  @[SerialVersionUID](https://www.scala-lang.org/api/2.13.x/scala/SerialVersionUID.html)()

- Source

  [Regex.scala](https://github.com/scala/scala/tree/v2.13.8/src/library/scala/util/matching/Regex.scala#L207)

- See also

  [java.util.regex.Pattern](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)