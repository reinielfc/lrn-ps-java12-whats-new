# 2. Java 12 API Updates

## Catching Up

Java Release Schedule:

2 major releases every year, 1 LTS release every 3 years

- Java 9: introduced the module system
- Java 10: introduced local variable type inference, allows to replace the type of variable with just "var"
- Java 11: LTS

## New String Methods

### `String::indent`

```java
class MyClass {
    public static void main(String[] args) {
        String string = "hello".indent(2); // ==> "  hello\n"

        String multiline = "This\nis a\nmulti-line\nString";
        multiline.lines().forEach(System.out::println);
        /* OUTPUT:
         * This
         * is a
         * multi-line
         * String
         */

        String indented = multiline.indent(5);
        indented.lines().forEach(System.out::println);
        /* OUTPUT:
         *      This
         *      is a
         *      multi-line
         *      String
         */

        indented.indent(-3).lines().forEach(System.out::println);
        /* OUTPUT:
         *   This
         *   is a
         *   multi-line
         *   String
         */
    }
}
```

### `String::transform`

```java
class StringUtils {
    public static String clean(String s) {
        return s.replaceAll("\\$", "");
    }

    public static String[] words(String s) {
        return s.split(" ");
    }
}

public class MyClass {
    public static void main(String[] args) {
        var text = "$some text $with$ $dollars";

        // before transform
        var cleanText = StringUtils.clean(text); // ==> "some text with dollars"
        String[] words = StringUtils.words(cleanText); // ==> { "some", "text", "with", "dollars" }
        // too many intermediate variables

        // or alternatively
        words = StringUtils.words(StringUtils.clean(text)); // less readable

        // with transform
        words = text
                .transform(StringUtils::clean)
                .transform(StringUtils::words);

        // add as many transformations as needed
        String[] upperCaseWords = text.transform(String::toUpperCase)
                .transform(StringUtils::clean)
                .transform(StringUtils::words); // ==> { "SOME", "TEXT", "WITH", "DOLLARS" }        
    }
}
```

## CompactNumberFormat

| Number  | US  | Germany |
|---------|-----|---------|
| 1000    | 1K  | 1.000   |
| 1000000 | 1M  | 1 Mio.  |

```java
import java.math.RoundingMode;
import java.text.CompactNumberFormat;
import java.text.NumberFormat;
import java.util.Locale;

class MyClass {
    public static void main(String[] args) {
        NumberFormat shortNF = NumberFormat.getCompactNumberInstance();
        shortNF.format(1000); // => "1K"
        shortNF.format(1_000_000); // => "1M" // the _s make the numbers more readable (feature from Java 7)
        shortNF.format(1500); // ==> "2K" // rounds up, can be changed with NumberFormat::setRoundingMode

        shortNF = NumberFormat.getCompactNumberInstance();
        shortNF.setRoundingMode(RoundingMode.DOWN);
        shortNF.format(1500); // ==> "1K"

        // set faction digits
        shortNF = NumberFormat.getCompactNumberInstance();
        shortNF.setMaximumFractionDigits(2);
        shortNF.format(1500); // ==> "1.5K"

        // set locale
        shortNF = NumberFormat.getCompactNumberInstance(
                Locale.GERMAN, NumberFormat.Style.SHORT);
        shortNF.format(1_000_000); // ==> "1 Mio."
    }
}
```

You can also create your own custom patterns:

```
CompactNumberFormat(
    String decimalPattern,
    DecimalFormatSymbols symbols,
    String[] compactPatterns)
```

`compactPatterns` array example:

```
{"", "", "", "0K", "00K", "000K", "0M", ... "0B" ... "0T" }
```

## Streams: The Teeing Collector

### Collector

Applies a single transformation to the stream and returns a single result

Stream.of(...) ==> .collect(Collector) (eg. Collectors.toList(), .toSet(), .counting())

### Teeing Collector

Allows you to do 2 things at once in the collection phase

Stream.of(...) ==> .collect(Collectors.teeing(
--> collector1,
--> collector2,
--> combineFunction)) ==> singleResult

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

class MyClass {
    public static void main(String[] args) {
        var ints = Stream.of(10, 20, 30, 40);
        long average = ints.collect(
                Collectors.teeing(
                        Collectors.summingInt(Integer::valueOf), // sum of all elements (100)
                        Collectors.counting(),                   // count of all elements (4)
                        (sum, count) -> sum / count              // average (25)
                )
        ); // ==> 25L
    }
}
```

## `Files::mismatch`

`Files.mismatch(Path.of("/file1"), Path.of("/file2"));`

- compares the bytes in the files
- if equal, returns -1
- if not, returns location of first mismatching byte
