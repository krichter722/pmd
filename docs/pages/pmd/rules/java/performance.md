---
title: Performance
summary: Rules that flag suboptimal code.
permalink: pmd_rules_java_performance.html
folder: pmd/rules/java
sidebaractiveurl: /pmd_rules_java.html
editmepath: ../pmd-java/src/main/resources/category/java/performance.xml
keywords: Performance, AddEmptyString, AppendCharacterWithChar, AvoidArrayLoops, AvoidFileStream, AvoidInstantiatingObjectsInLoops, AvoidUsingShortType, BigIntegerInstantiation, BooleanInstantiation, ByteInstantiation, ConsecutiveAppendsShouldReuse, ConsecutiveLiteralAppends, InefficientEmptyStringCheck, InefficientStringBuffering, InsufficientStringBufferDeclaration, IntegerInstantiation, LongInstantiation, OptimizableToArrayCall, RedundantFieldInitializer, SimplifyStartsWith, ShortInstantiation, StringInstantiation, StringToString, TooFewBranchesForASwitchStatement, UnnecessaryWrapperObjectCreation, UseArrayListInsteadOfVector, UseArraysAsList, UseIndexOfChar, UselessStringValueOf, UseStringBufferForStringAppends, UseStringBufferLength
language: Java
---
## AddEmptyString

**Since:** PMD 4.0

**Priority:** Medium (3)

The conversion of literals to strings by concatenating them with empty strings is inefficient.
It is much better to use one of the type-specific toString() methods instead.

**This rule is defined by the following XPath expression:**
```
//AdditiveExpression/PrimaryExpression/PrimaryPrefix/Literal[@Image='""']
```

**Example(s):**

``` java
String s = "" + 123;                // inefficient
String t = Integer.toString(456);   // preferred approach
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/AddEmptyString" />
```

## AppendCharacterWithChar

**Since:** PMD 3.5

**Priority:** Medium (3)

Avoid concatenating characters as strings in StringBuffer/StringBuilder.append methods.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.AppendCharacterWithCharRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/AppendCharacterWithCharRule.java)

**Example(s):**

``` java
StringBuffer sb = new StringBuffer();
sb.append("a");     // avoid this

StringBuffer sb = new StringBuffer();
sb.append('a');     // use this instead
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/AppendCharacterWithChar" />
```

## AvoidArrayLoops

**Since:** PMD 3.5

**Priority:** Medium (3)

Instead of manually copying data between two arrays, use the efficient Arrays.copyOf or System.arraycopy method instead.

**This rule is defined by the following XPath expression:**
```
//Statement[(ForStatement or WhileStatement) and
count(*//AssignmentOperator[@Image = '='])=1
and
*/Statement
[
./Block/BlockStatement/Statement/StatementExpression/PrimaryExpression
/PrimaryPrefix/Name/../../PrimarySuffix/Expression
[(PrimaryExpression or AdditiveExpression) and count
(.//PrimaryPrefix/Name)=1]//PrimaryPrefix/Name/@Image
and
./Block/BlockStatement/Statement/StatementExpression/Expression/PrimaryExpression
/PrimaryPrefix/Name/../../PrimarySuffix[count
(..//PrimarySuffix)=1]/Expression[(PrimaryExpression
or AdditiveExpression) and count(.//PrimaryPrefix/Name)=1]
//PrimaryPrefix/Name/@Image
]]
```

**Example(s):**

``` java
public class Test {
    public void bar() {
        int[] a = new int[10];
        int[] b = new int[10];
        for (int i=0;i<10;i++) {
            b[i]=a[i];
        }

        int[] c = new int[10];
        // this will trigger the rule
        for (int i=0;i<10;i++) {
            b[i]=a[c[i]];
        }
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/AvoidArrayLoops" />
```

## AvoidFileStream

**Since:** PMD 6.0.0

**Priority:** High (1)

**Minimum Language Version:** Java 1.7

The FileInputStream and FileOutputStream classes contains a finalizer method which will cause garbage collection pauses. See [JDK-8080225](https://bugs.openjdk.java.net/browse/JDK-8080225) for details.

The FileReader and FileWriter constructors instantiate FileInputStream and FileOutputStream, again causing garbage collection issues while finalizer methods are called.

* Use `Files.newInputStream(Paths.get(fileName))` instead of `new FileInputStream(fileName)`.
* Use `Files.newOutputStream(Paths.get(fileName))` instead of `new FileOutputStream(fileName)`.
* Use `Files.newBufferedReader(Paths.get(fileName))` instead of `new FileReader(fileName)`.
* Use `Files.newBufferedWriter(Paths.get(fileName))` instead of `new FileWriter(fileName)`.

**This rule is defined by the following XPath expression:**
```
//PrimaryPrefix/AllocationExpression/ClassOrInterfaceType[
       typeof(@Image, 'java.io.FileInputStream', 'FileInputStream')
    or typeof(@Image, 'java.io.FileOutputStream', 'FileOutputStream')
    or typeof(@Image, 'java.io.FileReader', 'FileReader')
    or typeof(@Image, 'java.io.FileWriter', 'FileWriter')
  ]
```

**Example(s):**

``` java
// these instantiations cause garbage collection pauses, even if properly closed

    FileInputStream fis = new FileInputStream(fileName);
    FileOutputStream fos = new FileOutputStream(fileName);
    FileReader fr = new FileReader(fileName);
    FileWriter fw = new FileWriter(fileName);

    // the following instantiations help prevent Garbage Collection pauses, no finalization

    try(InputStream is = Files.newInputStream(Paths.get(fileName))) {
    }
    try(OutputStream os = Files.newOutputStream(Paths.get(fileName))) {
    }
    try(BufferedReader br = Files.newBufferedReader(Paths.get(fileName), StandardCharsets.UTF_8)) {
    }
    try(BufferedWriter wr = Files.newBufferedWriter(Paths.get(fileName), StandardCharsets.UTF_8)) {
    }
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/AvoidFileStream" />
```

## AvoidInstantiatingObjectsInLoops

**Since:** PMD 2.2

**Priority:** Medium (3)

New objects created within loops should be checked to see if they can created outside them and reused.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.AvoidInstantiatingObjectsInLoopsRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/AvoidInstantiatingObjectsInLoopsRule.java)

**Example(s):**

``` java
public class Something {
    public static void main( String as[] ) {
        for (int i = 0; i < 10; i++) {
            Foo f = new Foo(); // Avoid this whenever you can it's really expensive
        }
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/AvoidInstantiatingObjectsInLoops" />
```

## AvoidUsingShortType

**Since:** PMD 4.1

**Priority:** High (1)

Java uses the 'short' type to reduce memory usage, not to optimize calculation. In fact, the JVM does not have any
arithmetic capabilities for the short type: the JVM must convert the short into an int, do the proper calculation
and convert the int back to a short. Thus any storage gains found through use of the 'short' type may be offset by
adverse impacts on performance.

**This rule is defined by the following XPath expression:**
```
//FieldDeclaration/Type/PrimitiveType[@Image = 'short']
|
//ClassOrInterfaceBodyDeclaration[not(Annotation/MarkerAnnotation/Name[typeof(@Image, 'java.lang.Override', 'Override')])]
    /MethodDeclaration/ResultType/Type/PrimitiveType[@Image = 'short']
|
//ClassOrInterfaceBodyDeclaration[not(Annotation/MarkerAnnotation/Name[typeof(@Image, 'java.lang.Override', 'Override')])]
    /MethodDeclaration/MethodDeclarator/FormalParameters/FormalParameter/Type/PrimitiveType[@Image = 'short']
|
//LocalVariableDeclaration/Type/PrimitiveType[@Image = 'short']
|
//AnnotationMethodDeclaration/Type/PrimitiveType[@Image = 'short']
```

**Example(s):**

``` java
public class UsingShort {
   private short doNotUseShort = 0;

   public UsingShort() {
    short shouldNotBeUsed = 1;
    doNotUseShort += shouldNotBeUsed;
  }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/AvoidUsingShortType" />
```

## BigIntegerInstantiation

**Since:** PMD 3.9

**Priority:** Medium (3)

Don't create instances of already existing BigInteger (BigInteger.ZERO, BigInteger.ONE) and
for Java 1.5 onwards, BigInteger.TEN and BigDecimal (BigDecimal.ZERO, BigDecimal.ONE, BigDecimal.TEN)

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.BigIntegerInstantiationRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/BigIntegerInstantiationRule.java)

**Example(s):**

``` java
BigInteger bi = new BigInteger(1);       // reference BigInteger.ONE instead
BigInteger bi2 = new BigInteger("0");    // reference BigInteger.ZERO instead
BigInteger bi3 = new BigInteger(0.0);    // reference BigInteger.ZERO instead
BigInteger bi4;
bi4 = new BigInteger(0);                 // reference BigInteger.ZERO instead
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/BigIntegerInstantiation" />
```

## BooleanInstantiation

**Since:** PMD 1.2

**Priority:** Medium High (2)

Avoid instantiating Boolean objects; you can reference Boolean.TRUE, Boolean.FALSE, or call Boolean.valueOf() instead.
Note that new Boolean() is deprecated since JDK 9 for that reason.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.BooleanInstantiationRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/BooleanInstantiationRule.java)

**Example(s):**

``` java
Boolean bar = new Boolean("true");        // unnecessary creation, just reference Boolean.TRUE;
Boolean buz = Boolean.valueOf(false);    // ...., just reference Boolean.FALSE;
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/BooleanInstantiation" />
```

## ByteInstantiation

**Since:** PMD 4.0

**Priority:** Medium High (2)

Calling new Byte() causes memory allocation that can be avoided by the static Byte.valueOf().
It makes use of an internal cache that recycles earlier instances making it more memory efficient.
Note that new Byte() is deprecated since JDK 9 for that reason.

**This rule is defined by the following XPath expression:**
```
//AllocationExpression
[not (ArrayDimsAndInits)
and ClassOrInterfaceType[typeof(@Image, 'java.lang.Byte', 'Byte')]]
```

**Example(s):**

``` java
public class Foo {
    private Byte i = new Byte(0); // change to Byte i = Byte.valueOf(0);
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/ByteInstantiation" />
```

## ConsecutiveAppendsShouldReuse

**Since:** PMD 5.1

**Priority:** Medium (3)

Consecutive calls to StringBuffer/StringBuilder .append should be chained, reusing the target object. This can improve the performance
by producing a smaller bytecode, reducing overhead and improving inlining. A complete analysis can be found [here](https://github.com/pmd/pmd/issues/202#issuecomment-274349067)

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.ConsecutiveAppendsShouldReuseRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/ConsecutiveAppendsShouldReuseRule.java)

**Example(s):**

``` java
String foo = " ";

StringBuffer buf = new StringBuffer();
buf.append("Hello"); // poor
buf.append(foo);
buf.append("World");

StringBuffer buf = new StringBuffer();
buf.append("Hello").append(foo).append("World"); // good
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/ConsecutiveAppendsShouldReuse" />
```

## ConsecutiveLiteralAppends

**Since:** PMD 3.5

**Priority:** Medium (3)

Consecutively calling StringBuffer/StringBuilder.append(...) with literals should be avoided.
Since the literals are constants, they can already be combined into a single String literal and this String
can be appended in a single method call.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.ConsecutiveLiteralAppendsRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/ConsecutiveLiteralAppendsRule.java)

**Example(s):**

``` java
StringBuilder buf = new StringBuilder();
buf.append("Hello").append(" ").append("World");    // poor
buf.append("Hello World");                          // good

buf.append('h').append('e').append('l').append('l').append('o'); // poor
buf.append("hello");                                             // good

buf.append(1).append('m');  // poor
buf.append("1m");           // good
```

**This rule has the following properties:**

|Name|Default Value|Description|Multivalued|
|----|-------------|-----------|-----------|
|threshold|1|Max consecutive appends|no|

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/ConsecutiveLiteralAppends" />
```

## InefficientEmptyStringCheck

**Since:** PMD 3.6

**Priority:** Medium (3)

String.trim().length() is an inefficient way to check if a String is really empty, as it
creates a new String object just to check its size. Consider creating a static function that
loops through a string, checking Character.isWhitespace() on each character and returning
false if a non-whitespace character is found. You can refer to Apache's StringUtils#isBlank (in commons-lang),
Spring's StringUtils#hasText (in the Spring framework) or Google's CharMatcher#whitespace (in Guava) for
existing implementations.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.InefficientEmptyStringCheckRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/InefficientEmptyStringCheckRule.java)

**Example(s):**

``` java
public void bar(String string) {
    if (string != null && string.trim().size() > 0) {
        doSomething();
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/InefficientEmptyStringCheck" />
```

## InefficientStringBuffering

**Since:** PMD 3.4

**Priority:** Medium (3)

Avoid concatenating non-literals in a StringBuffer constructor or append() since intermediate buffers will
need to be be created and destroyed by the JVM.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.InefficientStringBufferingRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/InefficientStringBufferingRule.java)

**Example(s):**

``` java
// Avoid this, two buffers are actually being created here
StringBuffer sb = new StringBuffer("tmp = "+System.getProperty("java.io.tmpdir"));

// do this instead
StringBuffer sb = new StringBuffer("tmp = ");
sb.append(System.getProperty("java.io.tmpdir"));
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/InefficientStringBuffering" />
```

## InsufficientStringBufferDeclaration

**Since:** PMD 3.6

**Priority:** Medium (3)

Failing to pre-size a StringBuffer or StringBuilder properly could cause it to re-size many times
during runtime. This rule attempts to determine the total number the characters that are actually 
passed into StringBuffer.append(), but represents a best guess "worst case" scenario. An empty
StringBuffer/StringBuilder constructor initializes the object to 16 characters. This default
is assumed if the length of the constructor can not be determined.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.InsufficientStringBufferDeclarationRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/InsufficientStringBufferDeclarationRule.java)

**Example(s):**

``` java
StringBuffer bad = new StringBuffer();
bad.append("This is a long string that will exceed the default 16 characters");

StringBuffer good = new StringBuffer(41);
good.append("This is a long string, which is pre-sized");
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/InsufficientStringBufferDeclaration" />
```

## IntegerInstantiation

**Since:** PMD 3.5

**Priority:** Medium High (2)

Calling new Integer() causes memory allocation that can be avoided by the static Integer.valueOf().
It makes use of an internal cache that recycles earlier instances making it more memory efficient.
Note that new Integer() is deprecated since JDK 9 for that reason.

**This rule is defined by the following XPath expression:**
```
//AllocationExpression
  [not (ArrayDimsAndInits)
   and ClassOrInterfaceType[typeof(@Image, 'java.lang.Integer', 'Integer')]]
```

**Example(s):**

``` java
public class Foo {
    private Integer i = new Integer(0); // change to Integer i = Integer.valueOf(0);
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/IntegerInstantiation" />
```

## LongInstantiation

**Since:** PMD 4.0

**Priority:** Medium High (2)

Calling new Long() causes memory allocation that can be avoided by the static Long.valueOf().
It makes use of an internal cache that recycles earlier instances making it more memory efficient.
Note that new Long() is deprecated since JDK 9 for that reason.

**This rule is defined by the following XPath expression:**
```
//AllocationExpression
[not (ArrayDimsAndInits)
and ClassOrInterfaceType[typeof(@Image, 'java.lang.Long', 'Long')]]
```

**Example(s):**

``` java
public class Foo {
    private Long i = new Long(0); // change to Long i = Long.valueOf(0);
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/LongInstantiation" />
```

## OptimizableToArrayCall

**Since:** PMD 1.8

**Priority:** Medium (3)

**Minimum Language Version:** Java 1.6

Calls to a collection's `toArray(E[])` method should specify a target array of zero size. This allows the JVM
to optimize the memory allocation and copying as much as possible.

Previous versions of this rule (pre PMD 6.0.0) suggested the opposite, but current JVM implementations
perform always better, when they have full control over the target array. And allocation an array via
reflection is nowadays as fast as the direct allocation.

See also [Arrays of Wisdom of the Ancients](https://shipilev.net/blog/2016/arrays-wisdom-ancients/)

Note: If you don't need an array of the correct type, then the simple `toArray()` method without an array
is faster, but returns only an array of type `Object[]`.

**This rule is defined by the following XPath expression:**
```
//PrimaryExpression
[PrimaryPrefix/Name[ends-with(@Image, 'toArray')]]
[
PrimarySuffix/Arguments/ArgumentList/Expression
 /PrimaryExpression/PrimaryPrefix/AllocationExpression
 /ArrayDimsAndInits/Expression/PrimaryExpression/PrimaryPrefix[not(Literal[@Image='0'])]
]
```

**Example(s):**

``` java
List<Foo> foos = getFoos();

// much better; this one allows the jvm to allocate an array of the correct size and effectively skip
// the zeroing, since each array element will be overridden anyways
Foo[] fooArray = foos.toArray(new Foo[0]);

// inefficient, the array needs to be zeroed out by the jvm before it is handed over to the toArray method
Foo[] fooArray = foos.toArray(new Foo[foos.size()]);
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/OptimizableToArrayCall" />
```

## RedundantFieldInitializer

**Since:** PMD 5.0

**Priority:** Medium (3)

Java will initialize fields with known default values so any explicit initialization of those same defaults
is redundant and results in a larger class file (approximately three additional bytecode instructions per field).

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.RedundantFieldInitializerRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/RedundantFieldInitializerRule.java)

**Example(s):**

``` java
public class C {
    boolean b   = false;    // examples of redundant initializers
    byte by     = 0;
    short s     = 0;
    char c      = 0;
    int i       = 0;
    long l      = 0;

    float f     = .0f;    // all possible float literals
    doable d    = 0d;     // all possible double literals
    Object o    = null;

    MyClass mca[] = null;
    int i1 = 0, ia1[] = null;

    class Nested {
        boolean b = false;
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/RedundantFieldInitializer" />
```

## ShortInstantiation

**Since:** PMD 4.0

**Priority:** Medium High (2)

Calling new Short() causes memory allocation that can be avoided by the static Short.valueOf().
It makes use of an internal cache that recycles earlier instances making it more memory efficient.
Note that new Short() is deprecated since JDK 9 for that reason.

**This rule is defined by the following XPath expression:**
```
//AllocationExpression
[not (ArrayDimsAndInits)
and ClassOrInterfaceType[typeof(@Image, 'java.lang.Short', 'Short')]]
```

**Example(s):**

``` java
public class Foo {
    private Short i = new Short(0); // change to Short i = Short.valueOf(0);
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/ShortInstantiation" />
```

## SimplifyStartsWith

**Since:** PMD 3.1

**Priority:** Medium (3)

Since it passes in a literal of length 1, calls to (string).startsWith can be rewritten using (string).charAt(0)
at the expense of some readability.

**This rule is defined by the following XPath expression:**
```
//PrimaryExpression
 [PrimaryPrefix/Name
  [ends-with(@Image, '.startsWith')] or PrimarySuffix[@Image='startsWith']]
 [PrimarySuffix/Arguments/ArgumentList
  /Expression/PrimaryExpression/PrimaryPrefix
  /Literal
   [string-length(@Image)=3]
   [starts-with(@Image, '"')]
   [ends-with(@Image, '"')]
 ]
```

**Example(s):**

``` java
public class Foo {

    boolean checkIt(String x) {
        return x.startsWith("a");   // suboptimal
    }

    boolean fasterCheckIt(String x) {
        return x.charAt(0) == 'a';  // faster approach
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/SimplifyStartsWith" />
```

## StringInstantiation

**Since:** PMD 1.0

**Priority:** Medium High (2)

Avoid instantiating String objects; this is usually unnecessary since they are immutable and can be safely shared.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.StringInstantiationRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/StringInstantiationRule.java)

**Example(s):**

``` java
private String bar = new String("bar"); // just do a String bar = "bar";
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/StringInstantiation" />
```

## StringToString

**Since:** PMD 1.0

**Priority:** Medium (3)

Avoid calling toString() on objects already known to be string instances; this is unnecessary.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.StringToStringRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/StringToStringRule.java)

**Example(s):**

``` java
private String baz() {
    String bar = "howdy";
    return bar.toString();
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/StringToString" />
```

## TooFewBranchesForASwitchStatement

**Since:** PMD 4.2

**Priority:** Medium (3)

Switch statements are intended to be used to support complex branching behaviour. Using a switch for only a few
cases is ill-advised, since switches are not as easy to understand as if-then statements. In these cases use the
if-then statement to increase code readability.

**This rule is defined by the following XPath expression:**
```
//SwitchStatement[
    (count(.//SwitchLabel) < $minimumNumberCaseForASwitch)
]
```

**Example(s):**

``` java
// With a minimumNumberCaseForASwitch of 3
public class Foo {
    public void bar() {
        switch (condition) {
            case ONE:
                instruction;
                break;
            default:
                break; // not enough for a 'switch' stmt, a simple 'if' stmt would have been more appropriate
        }
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|Multivalued|
|----|-------------|-----------|-----------|
|minimumNumberCaseForASwitch|3|Minimum number of branches for a switch|no|

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/TooFewBranchesForASwitchStatement" />
```

## UnnecessaryWrapperObjectCreation

**Since:** PMD 3.8

**Priority:** Medium (3)

Most wrapper classes provide static conversion methods that avoid the need to create intermediate objects
just to create the primitive forms. Using these avoids the cost of creating objects that also need to be 
garbage-collected later.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.UnnecessaryWrapperObjectCreationRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/UnnecessaryWrapperObjectCreationRule.java)

**Example(s):**

``` java
public int convert(String s) {
    int i, i2;

    i = Integer.valueOf(s).intValue();  // this wastes an object
    i = Integer.parseInt(s);            // this is better

    i2 = Integer.valueOf(i).intValue(); // this wastes an object
    i2 = i;                             // this is better

    String s3 = Integer.valueOf(i2).toString(); // this wastes an object
    s3 = Integer.toString(i2);                  // this is better

    return i2;
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/UnnecessaryWrapperObjectCreation" />
```

## UseArrayListInsteadOfVector

**Since:** PMD 3.0

**Priority:** Medium (3)

ArrayList is a much better Collection implementation than Vector if thread-safe operation is not required.

**This rule is defined by the following XPath expression:**
```
//CompilationUnit[count(ImportDeclaration) = 0 or count(ImportDeclaration/Name[@Image='java.util.Vector']) > 0]
  //AllocationExpression/ClassOrInterfaceType
    [@Image='Vector' or @Image='java.util.Vector']
```

**Example(s):**

``` java
public class SimpleTest extends TestCase {
    public void testX() {
    Collection c1 = new Vector();
    Collection c2 = new ArrayList();    // achieves the same with much better performance
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/UseArrayListInsteadOfVector" />
```

## UseArraysAsList

**Since:** PMD 3.5

**Priority:** Medium (3)

 
The java.util.Arrays class has a "asList" method that should be used when you want to create a new List from
an array of objects. It is faster than executing a loop to copy all the elements of the array one by one.

Note that the result of Arrays.asList() is backed by the specified array,
changes in the returned list will result in the array to be modified.
For that reason, it is not possible to add new elements to the returned list of Arrays.asList() (UnsupportedOperationException).
You must use new ArrayList<>(Arrays.asList(...)) if that is inconvenient for you (e.g. because of concurrent access).

	

**This rule is defined by the following XPath expression:**
```
//Statement[
    (ForStatement) and (ForStatement//VariableInitializer//Literal[@IntLiteral='true' and @Image='0']) and (count(.//IfStatement)=0)
   ]
   //StatementExpression[
    PrimaryExpression/PrimaryPrefix/Name[
     substring-before(@Image,'.add') = ancestor::MethodDeclaration//LocalVariableDeclaration[
      ./Type//ClassOrInterfaceType[
       @Image = 'Collection' or 
       @Image = 'List' or @Image='ArrayList'
      ]
     ]
     /VariableDeclarator/VariableDeclaratorId[
      count(..//AllocationExpression/ClassOrInterfaceType[
       @Image="ArrayList"
      ]
      )=1
     ]/@Image
    ]
   and
   PrimaryExpression/PrimarySuffix/Arguments/ArgumentList/Expression/PrimaryExpression/PrimaryPrefix/Name
   [
     @Image = ancestor::MethodDeclaration//LocalVariableDeclaration[@Array="true"]/VariableDeclarator/VariableDeclaratorId/@Image
     or
     @Image = ancestor::MethodDeclaration//FormalParameter/VariableDeclaratorId/@Image
   ]
   /../..[count(.//PrimarySuffix)
   =1]/PrimarySuffix/Expression/PrimaryExpression/PrimaryPrefix
   /Name
   ]
```

**Example(s):**

``` java
public class Test {
    public void foo(Integer[] ints) {
        // could just use Arrays.asList(ints)
        List<Integer> l= new ArrayList<>(100);
        for (int i=0; i< 100; i++) {
            l.add(ints[i]);
        }
        for (int i=0; i< 100; i++) {
            l.add(a[i].toString()); // won't trigger the rule
        }
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/UseArraysAsList" />
```

## UseIndexOfChar

**Since:** PMD 3.5

**Priority:** Medium (3)

Use String.indexOf(char) when checking for the index of a single character; it executes faster.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.UseIndexOfCharRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/UseIndexOfCharRule.java)

**Example(s):**

``` java
String s = "hello world";
// avoid this
if (s.indexOf("d") {}
// instead do this
if (s.indexOf('d') {}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/UseIndexOfChar" />
```

## UselessStringValueOf

**Since:** PMD 3.8

**Priority:** Medium (3)

No need to call String.valueOf to append to a string; just use the valueOf() argument directly.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.UselessStringValueOfRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/UselessStringValueOfRule.java)

**Example(s):**

``` java
public String convert(int i) {
    String s;
    s = "a" + String.valueOf(i);    // not required
    s = "a" + i;                    // preferred approach
    return s;
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/UselessStringValueOf" />
```

## UseStringBufferForStringAppends

**Since:** PMD 3.1

**Priority:** Medium (3)

The use of the '+=' operator for appending strings causes the JVM to create and use an internal StringBuffer.
If a non-trivial number of these concatenations are being used then the explicit use of a StringBuilder or 
threadsafe StringBuffer is recommended to avoid this.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.UseStringBufferForStringAppendsRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/UseStringBufferForStringAppendsRule.java)

**Example(s):**

``` java
public class Foo {
    void bar() {
        String a;
        a = "foo";
        a += " bar";
        // better would be:
        // StringBuilder a = new StringBuilder("foo");
        // a.append(" bar);
    }
}
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/UseStringBufferForStringAppends" />
```

## UseStringBufferLength

**Since:** PMD 3.4

**Priority:** Medium (3)

Use StringBuffer.length() to determine StringBuffer length rather than using StringBuffer.toString().equals("")
or StringBuffer.toString().length() == ...

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.java.rule.performance.UseStringBufferLengthRule](https://github.com/pmd/pmd/blob/master/pmd-java/src/main/java/net/sourceforge/pmd/lang/java/rule/performance/UseStringBufferLengthRule.java)

**Example(s):**

``` java
StringBuffer sb = new StringBuffer();

if (sb.toString().equals("")) {}        // inefficient

if (sb.length() == 0) {}                // preferred
```

**Use this rule by referencing it:**
``` xml
<rule ref="category/java/performance.xml/UseStringBufferLength" />
```

