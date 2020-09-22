https://github.com/google/styleguide  
https://google.github.io/styleguide/javaguide.html  

主要是针对我平时没注意的地方整理了一下，平时无事后者code clean时候可以回顾，并修订Intelji的验证机制。

3.3.1.Wildcard imports, static or otherwise, are not used. 不要用wildcard引入包，而且import不换行

3.3.4.Static import is not used for static nested classes. They are imported with normal imports.   不要引入static内部类，使用通常的引用import  

4.1.2.Nonempty blocks: K & R style
```
Examples:

return () -> {
  while (condition()) {
    method();
  }
};
//这个@Override的位置比较有趣
return new MyClass() {
  @Override public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    } else if (otherCondition()) {
      somethingElse();
    } else {
      lastThing();
    }
  }
};
```

4.1.3.Empty blocks: may be concise  
```
// This is acceptable
void doNothing() {}

// This is equally acceptable
void doNothingElse() {
}

// This is not acceptable: No concise empty blocks in a multi-block statement
try {
  doSomething();
} catch (Exception e) {}
```

4.2.Block indentation: +2 spaces 关于缩进，google建议是2个空格

4.4.Column limit: 100，大部分情况下每行最多100字符
```
Exceptions:
1.Lines where obeying the column limit is not possible (for example, a long URL in Javadoc, or a long JSNI method reference).
不可能满足列限制的行(例如，Javadoc中的一个长URL，或是一个长的JSNI方法参考)。
2.package and import statements (see Sections 3.2 Package statement and 3.3 Import statements).
package和import语句(见章节3.2和章节3.3)。
3.Command lines in a comment that may be cut-and-pasted into a shell.
注释中那些可能被剪切并粘贴到shell中的命令行。
```

4.6.2.Horizontal whitespace，正常情况大括号{}都需要留个空，除非
```
@SomeAnnotation({a, b})（不使用空格）
String[][] x = {{"foo"}};（两个大括号之间无空格）

类型界限中的&符：<T extends Foo & Bar>
catch块中的管道符号：catch (FooException | BarException e)
```

4.6.3.Horizontal alignment: never required 不需要垂直对应变量和comment
```
private int x; // this is fine
private Color color; // this too

private int   x;      // permitted, but future edits
private Color color;  // may leave it unaligned
```

4.8.2.1.每次只声明一个变量，不要使用组合声明，如int a, b;  

4.8.2.2.需要时才声明，并尽快进行初始化  
不要在一个代码块的开头把局部变量一次性都声明了，而是在第一次需要使用它时才声明。局部变量在声明时最好就进行初始化，或者声明后尽快进行初始化

4.8.3.Arrays
```
Any array initializer may optionally be formatted as if it were a "block-like construct." For example, the following are all valid (not an exhaustive list):

new int[] {           new int[] {
  0, 1, 2, 3            0,
}                       1,
                        2,
new int[] {             3,
  0, 1,               }
  2, 3
}                     new int[]
                          {0, 1, 2, 3}
```

4.8.4.2.Fall-through：注释，主要用于switch case没有break的情况，加上fall through方便阅读  
```
switch (input) {
    case 1:
    case 2:
        prepareOneOrTwo();
        // fall through
    case 3:
        handleOneTwoOrThree();
        break;
    default:
        handleLargeNumber(input);
}
```

4.8.5.Annotations，正常情况都是一行一个annotation，除非就一个annotation  
Exception: A single parameterless annotation may instead appear together with the first line of the signature, for example:
```
@Override public int hashCode() { ... }
```

4.8.6.Comments，其实比较随意，下面这样都行
```
/*
 * This is          // And so           /* Or you can
 * okay.            // is this.          * even do this. */
 */
```
4.8.8.Numeric Literals，主要是long的情况比较特殊，因为l看着和1一样。  
long-valued integer literals use an uppercase L suffix, never lowercase (to avoid confusion with the digit 1). For example, 3000000000L rather than 3000000000l.  

5.2.1.Package names，包名全部小写，没有下划线    

5.2.3 Method names，通常就是驼峰命名法，方法名称通常是动词或动词短语。例如，sendMessage或stop。但是Junit test的命名比较灵活，可以下划线。  
Underscores may appear in JUnit test method names to separate logical components of the name, with each component written in lowerCamelCase. One typical pattern is <methodUnderTest>_<state>, for example pop_emptyStack. There is no One Correct Way to name test methods.  

5.2.4 Constant names，CONSTANT_CASE命名法，必须是static final，一些immutable的变量也应该如此  
```
// Constants
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final ImmutableMap<String, Integer> AGES = ImmutableMap.of("Ed", 35, "Ann", 32);
static final Joiner COMMA_JOINER = Joiner.on(','); // because Joiner is immutable
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }

// Not constants
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final ImmutableMap<String, SomeMutableType> mutableValues =
    ImmutableMap.of("Ed", mutableInstance, "Ann", mutableInstance2);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```

5.3.Camel case: defined，驼峰命名法细节，主要针对个别特殊，比如"IPv6" or "iOS"的处理
1)将短语转换为纯ASCII并删除任何省略号。例如：“Müller's algorithm”可改为“Muellers algorithm”。  
2)以空格和任何剩余的标点符号（通常为连字符）将结果划分为单词。  
推荐： 如果有任何词在常用的情况下已经具有常规的CamelCase外观，将其拆分为其组成部分（例如，“AdWords”成为“ad words”）。 注意，诸如“iOS”这样的词本身不是真正的骆驼情况；它违反任何惯例，因此本建议不适用。  
3)将单词（包括首字母缩略词）第一个字符大写其他字符全小写：  
...将所有字符全大写  
...除第一个字符外，将所有字符全小写  
4)最后，将所有单词连接成一个词。  

注意，原始单词样式几乎完全被忽略。示例    
Prose form              Correct             Incorrect  
"XML HTTP request"	    XmlHttpRequest	    XMLHTTPRequest  
"new customer ID"	    newCustomerId	    newCustomerID  
"inner stopwatch"	    innerStopwatch	    innerStopWatch  
"supports IPv6 on iOS?"	supportsIpv6OnIos	supportsIPv6OnIOS  
"YouTube importer"	    YouTubeImporter  

6.1.@Override: always used，总是加上@Override如果是override的  

6.2.Caught exceptions: not ignored，空的catch必须有comment，除非是test case只要是expect exception  
When it truly is appropriate to take no action whatsoever in a catch block, the reason this is justified is explained in a comment.  
```
try {
  int i = Integer.parseInt(response);
  return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  // it's not numeric; that's fine, just continue
}
return handleTextResponse(response);
```
Exception: In tests, a caught exception may be ignored without comment if its name is or begins with expected. The following is a very common idiom for ensuring that the code under test does throw an exception of the expected type, so a comment is unnecessary here.  
```
try {
  emptyStack.pop();
  fail();
} catch (NoSuchElementException expected) {
}
```

6.3.Static members: qualified using class，静态方法就直接调用方法，不要用new的那个
```
Foo aFoo = ...;
Foo.aStaticMethod(); // good
aFoo.aStaticMethod(); // bad
somethingThatYieldsAFoo().aStaticMethod(); // very bad
```

6.4.Finalizers: not used，绝对不要用Finanlizers  
It is extremely rare to override Object.finalize.

7.1.1.General form  
The basic formatting of Javadoc blocks is as seen in this example:
```
/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String p1) { ... }
```
... or in this single-line example: 偷懒写法也接受
```
/** An especially short bit of Javadoc. */
```

7.2.摘要片段  
每个类或成员的Javadoc以一个简短的摘要片段开始。这个片段是非常重要的，在某些情况下，它是唯一出现的文本，比如在类和方法索引中。
这只是一个小片段，可以是一个名词短语或动词短语，但不是一个完整的句子。它不会以A {@code Foo} is a...或This method returns...开头,它也不会是一个完整的祈使句，如Save the record.。然而，由于开头大写及被加了标点，它看起来就像是个完整的句子。
```
提示： 一个常见的错误是把简单的Javadoc写成 /** @return the customer ID */，这是不正确的。
它应该写成/** Returns the customer ID. */。
```
这一部分不是很清楚，贴一个来源其他地方的格式
```
    /** 
     * Draws as much of the specified image as is currently available
     * with its northwest corner at the specified coordinate (x, y).
     * This method will return immediately in all cases, even if the
     * entire image has not yet been scaled, dithered and converted
     * for the current output device.
     * <p>
     * If the current output representation is not yet complete then
     * the method will return false and the indicated 
     * {@link ImageObserver} object will be notified as the
     * conversion process progresses.
     *
     * @param img the image to be drawn
     * @param x the x-coordinate of the northwest corner
     * of the destination rectangle in pixels
     * @param y the y-coordinate of the northwest corner
     * of the destination rectangle in pixels
     * @param observer the image observer to be notified as more
     * of the image is converted. May be 
     * <code>null</code>
     * @return <code>true</code> if the image is completely 
     * loaded and was painted successfully; 
     * <code>false</code> otherwise.
     * @see Image
     * @see ImageObserver
     * @since 1.0
     */
    public abstract boolean drawImage(Image img, int x, int y, 
                                      ImageObserver observer);
```
具体看[如何写出格式优美的javadoc？](https://www.jianshu.com/p/d9b10a3963ff)

7.3.Where Javadoc is used，至少每个publish class和protected class都需要javadoc  
At the minimum, Javadoc is present for every public class, and every public or protected member of such a class, with a few exceptions noted below.