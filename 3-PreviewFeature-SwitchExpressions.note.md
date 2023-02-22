## 3. Preview Feature: Switch Expressions

### Preview Features

- fully specified, fully implemented, and yet **impermanent** feature
- used to solicit feedback
- signal that the feature might change
- may be dropped
- similar to incubator modules for new APIs
- opt in with `--enable-preview` (compiler & runtime)

### The 'old' Switch Statement

```java
class Main {
    public static void main(String[] args) {
        int monthNumber = 11;
        switch (monthNumber) {
            case 1: System.out.println("January");
            //...
            case 11: System.out.println("November"); // OUTPUT: November
            case 12: System.out.println("December"); // OUTPUT: December
            default: System.out.println("Unknown"); // OUTPUT: Unknown
        }
        
        switch (monthNumber) {
            case 1: System.out.println("January"); break;
                //...
            case 11: System.out.println("November"); break; // OUTPUT: November
            case 12: System.out.println("December"); break;
            default: System.out.println("Unknown");
        }
        
        // no return value, the switch is a statement, not an expression
        // let's mimic a return value:
        String monthName;
        switch (monthNumber) {
            case 1: 
                String temp = "January"; // also, the switch statement doesn't introduce a new scope with each case
                monthName = temp; 
                break;
            //...
            case 11:
                //String temp = "November"; // this variable cannot be defined because it already exists 
                monthName = "November"; break;
            case 12: monthName = "December"; break;
            default: monthName = "Unknown";
        }
        System.out.println(monthName); // OUTPUT: November
        // not a nice workaround
    }
}
```

### Switch Expressions

- first ever java preview feature
- new switch variation
- doesn't replace the old one

```java
class Main {
    public static void main(String[] args) {
        int monthNumber = 11;
        String monthName = switch (monthNumber) { // becomes an expression, we can use it on the right hand side of an assignment
            case 1 -> "January"; // no breaks needed (no fallthrough)
            case 2 -> "February";
            case 3, 4, 5 -> "Spring months"; // combine case labels
            // ...
            case 11 -> { // blocks
                String month = "November";
                break month; // use break instead of return
            }
            case 12 -> {
                String month = "December"; // each case has its own scope, so we can reuse variable names
                break month;
            }
            default -> "Unknown";
        }; // ends with semicolon
        System.out.println(monthName); // OUTPUT: November
    }
}
```
