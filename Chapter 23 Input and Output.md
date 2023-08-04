### 23.1 — Input and output (I/O) streams

Input and output functionality is not defined as part of the core C++ language, but rather is provided through the C++ standard library (and thus resides in the std namespace). In previous lessons, you included the iostream library header and made use of the cin and cout objects to do simple I/O. In this lesson, we’ll take a look at the iostream library in more detail.

#### **The iostream library**

When you include the iostream header, you gain access to a whole hierarchy of classes responsible for providing I/O functionality (including one class that is actually named iostream). You can find a class hierarchy diagram for the non-file-I/O classes [here](https://en.cppreference.com/w/cpp/io).

The first thing you may notice about this hierarchy is that it uses multiple inheritance (that thing we told you to avoid if at all possible). However, the iostream library has been designed and extensively tested in order to avoid any of the typical multiple inheritance problems, so you can use it freely without worrying.

------

#### **Streams**

The second thing you may notice is that the word “stream” is used an awful lot. At its most basic, I/O in C++ is implemented with streams. Abstractly, a **stream** is just a sequence of bytes that can be accessed sequentially. Over time, a stream may produce or consume potentially unlimited amounts of data.

Typically we deal with two different types of streams. **Input streams** are used to hold input from a data producer, such as a keyboard, a file, or a network. For example, the user may press a key on the keyboard while the program is currently not expecting any input. Rather than ignore the users keypress, the data is put into an input stream, where it will wait until the program is ready for it.

Conversely, **output streams** are used to hold output for a particular data consumer, such as a monitor, a file, or a printer. When writing data to an output device, the device may not be ready to accept that data yet -- for example, the printer may still be warming up when the program writes data to its output stream. The data will sit in the output stream until the printer begins consuming it.

Some devices, such as files and networks, are capable of being both input and output sources.

The nice thing about streams is the programmer only has to learn how to interact with the streams in order to read and write data to many different kinds of devices. The details about how the stream interfaces with the actual devices they are hooked up to is left up to the environment or operating system.

------

#### **Input/output in C++**

Although the ios class is generally derived from ios_base, ios is typically the most base class you will be working directly with. The ios class defines a bunch of stuff that is common to both input and output streams. We’ll deal with this stuff in a future lesson.

The **istream** class is the primary class used when dealing with input streams. With input streams, the **extraction operator (>>)** is used to remove values from the stream. This makes sense: when the user presses a key on the keyboard, the key code is placed in an input stream. Your program then extracts the value from the stream so it can be used.

The **ostream** class is the primary class used when dealing with output streams. With output streams, the **insertion operator (<<)** is used to put values in the stream. This also makes sense: you insert your values into the stream, and the data consumer (e.g. monitor) uses them.

The **iostream** class can handle both input and output, allowing bidirectional I/O.

------

#### **Standard streams in C++**

A **standard stream** is a pre-connected stream provided to a computer program by its environment. C++ comes with four predefined standard stream objects that have already been set up for your use. The first three, you have seen before:

1. **cin** -- an istream object tied to the standard input (typically the keyboard)
2. **cout** -- an ostream object tied to the standard output (typically the monitor)
3. **cerr** -- an ostream object tied to the standard error (typically the monitor), providing unbuffered output
4. **clog** -- an ostream object tied to the standard error (typically the monitor), providing buffered output

Unbuffered output is typically handled immediately, whereas buffered output is typically stored and written out as a block. Because clog isn’t used very often, it is often omitted from the list of standard streams.

In the next lesson, we’ll take a look at some more I/O related functionality in more detail.

------

### 23.2 — Input with istream

The iostream library is fairly complex -- so we will not be able to cover it in its entirety in these tutorials. However, we will show you the most commonly used functionality. In this section, we will look at various aspects of the input class (istream).

#### **The extraction operator**

As seen in many lessons now, we can use the extraction operator (>>) to read information from an input stream. C++ has predefined extraction operations for all of the built-in data types, and you’ve already seen how you can [overload the extraction operator](https://www.learncpp.com/cpp-tutorial/93-overloading-the-io-operators/) for your own classes.

When reading strings, one common problem with the extraction operator is how to keep the input from overflowing your buffer. Given the following example:

```cpp
char buf[10];
std::cin >> buf;
```

what happens if the user enters 18 characters? The buffer overflows, and bad stuff happens. Generally speaking, it’s a bad idea to make any assumption about how many characters your user will enter.

One way to handle this problem is through use of manipulators. A **manipulator** is an object that is used to modify a stream when applied with the extraction (>>) or insertion (<<) operators. One manipulator you have already worked with is "std::endl", which both prints a newline character and flushes any buffered output. C++ provides a manipulator known as **setw** (in the iomanip header) that can be used to limit the number of characters read in from a stream. To use setw(), simply provide the maximum number of characters to read as a parameter, and insert it into your input statement like such:

```cpp
#include <iomanip>
char buf[10];
std::cin >> std::setw(10) >> buf;
```

This program will now only read the first 9 characters out of the stream (leaving room for a terminator). Any remaining characters will be left in the stream until the next extraction.

------

#### Extraction and whitespace

As a reminder, the extraction operator skips whitespace (blanks, tabs, and newlines).

Take a look at the following program:

```cpp
int main()
{
    char ch;
    while (std::cin >> ch)
        std::cout << ch;

    return 0;
}
```

When the user inputs the following:

```
Hello my name is Alex
```

The extraction operator skips the spaces and the newline. Consequently, the output is:

```
HellomynameisAlex
```

Oftentimes, you’ll want to get user input but not discard whitespace. To do this, the istream class provides many functions that can be used for this purpose.

One of the most useful is the **get()** function, which simply gets a character from the input stream. Here’s the same program as above using get():

```cpp
int main()
{
    char ch;
    while (std::cin.get(ch))
        std::cout << ch;

    return 0;
}
```

Now when we use the input:

```
Hello my name is Alex
```

The output is:

```
Hello my name is Alex
```

std::get() also has a string version that takes a maximum number of characters to read:

```cpp
int main()
{
    char strBuf[11];
    std::cin.get(strBuf, 11);
    std::cout << strBuf << '\n';

    return 0;
}
```

If we input:

```
Hello my name is Alex
```

The output is:

```
Hello my n
```

Note that we only read the first 10 characters (we had to leave one character for a terminator). The remaining characters were left in the input stream.

One important thing to note about get() is that it does not read in a newline character! This can cause some unexpected results:

```cpp
int main()
{
    char strBuf[11];
    // Read up to 10 characters
    std::cin.get(strBuf, 11);
    std::cout << strBuf << '\n';

    // Read up to 10 more characters
    std::cin.get(strBuf, 11);
    std::cout << strBuf << '\n';
    return 0;
}
```

If the user enters:

```
Hello!
```

The program will print:

```
Hello!
```

and then terminate! Why didn’t it ask for 10 more characters? The answer is because the first get() read up to the newline and then stopped. The second get() saw there was still input in the cin stream and tried to read it. But the first character was the newline, so it stopped immediately.

Consequently, there is another function called **getline()** that works exactly like get() but reads the newline as well.

```cpp
int main()
{
    char strBuf[11];
    // Read up to 10 characters
    std::cin.getline(strBuf, 11);
    std::cout << strBuf << '\n';

    // Read up to 10 more characters
    std::cin.getline(strBuf, 11);
    std::cout << strBuf << '\n';
    return 0;
}
```

This code will perform as you expect, even if the user enters a string with a newline in it.

If you need to know how many character were extracted by the last call of getline(), use **gcount()**:

```cpp
int main()
{
    char strBuf[100];
    std::cin.getline(strBuf, 100);
    std::cout << strBuf << '\n';
    std::cout << std::cin.gcount() << " characters were read" << '\n';

    return 0;
}
```

------

#### A special version of getline() for std::string

There is a special version of getline() that lives outside the istream class that is used for reading in variables of type std::string. This special version is not a member of either ostream or istream, and is included in the string header. Here is an example of its use:

```cpp
#include <string>
#include <iostream>

int main()
{
    std::string strBuf;
    std::getline(std::cin, strBuf);
    std::cout << strBuf << '\n';

    return 0;
}
```

------

#### A few more useful istream functions

There are a few more useful input functions that you might want to make use of:

**ignore()** discards the first character in the stream.
**ignore(int nCount)** discards the first nCount characters.
**peek()** allows you to read a character from the stream without removing it from the stream.
**unget()** returns the last character read back into the stream so it can be read again by the next call.
**putback(char ch)** allows you to put a character of your choice back into the stream to be read by the next call.

istream contains many other functions and variants of the above mentioned functions that may be useful, depending on what you need to do. You can find these on a reference site such as https://en.cppreference.com/w/cpp/io/basic_istream.

------

### 23.3 — Output with ostream and ios

In this section, we will look at various aspects of the iostream output class (ostream).

#### The insertion operator

The insertion operator (<<) is used to put information into an output stream. C++ has predefined insertion operations for all of the built-in data types, and you’ve already seen how you can [overload the insertion operator](https://www.learncpp.com/cpp-tutorial/93-overloading-the-io-operators/) for your own classes.

In the lesson on [streams](https://www.learncpp.com/cpp-tutorial/131-input-and-output-io-streams/), you saw that both istream and ostream were derived from a class called ios. One of the jobs of ios (and ios_base) is to control the formatting options for output.

------

#### Formatting

There are two ways to change the formatting options: flags, and manipulators. You can think of **flags** as boolean variables that can be turned on and off. **Manipulators** are objects placed in a stream that affect the way things are input and output.

To switch a flag on, use the **setf()** function, with the appropriate flag as a parameter. For example, by default, C++ does not print a + sign in front of positive numbers. However, by using the std::ios::showpos flag, we can change this behavior:

```cpp
std::cout.setf(std::ios::showpos); // turn on the std::ios::showpos flag
std::cout << 27 << '\n';
```

This results in the following output:

```
+27
```

It is possible to turn on multiple ios flags at once using the Bitwise OR (|) operator:

```cpp
std::cout.setf(std::ios::showpos | std::ios::uppercase); // turn on the std::ios::showpos and std::ios::uppercase flag
std::cout << 1234567.89f << '\n';
```

This outputs:

```
+1.23457E+06
```

To turn a flag off, use the **unsetf()** function:

```cpp
std::cout.setf(std::ios::showpos); // turn on the std::ios::showpos flag
std::cout << 27 << '\n';
std::cout.unsetf(std::ios::showpos); // turn off the std::ios::showpos flag
std::cout << 28 << '\n';
```

This results in the following output:

```
+27
28
```

There’s one other bit of trickiness when using setf() that needs to be mentioned. Many flags belong to groups, called format groups. A **format group** is a group of flags that perform similar (sometimes mutually exclusive) formatting options. For example, a format group named “basefield” contains the flags “oct”, “dec”, and “hex”, which controls the base of integral values. By default, the “dec” flag is set. Consequently, if we do this:

```cpp
std::cout.setf(std::ios::hex); // try to turn on hex output
std::cout << 27 << '\n';
```

We get the following output:

```
27
```

It didn’t work! The reason why is because setf() only turns flags on -- it isn’t smart enough to turn mutually exclusive flags off. Consequently, when we turned std::hex on, std::ios::dec was still on, and std::ios::dec apparently takes precedence. There are two ways to get around this problem.

First, we can turn off std::ios::dec so that only std::hex is set:

```cpp
std::cout.unsetf(std::ios::dec); // turn off decimal output
std::cout.setf(std::ios::hex); // turn on hexadecimal output
std::cout << 27 << '\n';
```

Now we get output as expected:

```
1b
```

The second way is to use a different form of setf() that takes two parameters: the first parameter is the flag to set, and the second is the formatting group it belongs to. When using this form of setf(), all of the flags belonging to the group are turned off, and only the flag passed in is turned on. For example:

```cpp
// Turn on std::ios::hex as the only std::ios::basefield flag
std::cout.setf(std::ios::hex, std::ios::basefield);
std::cout << 27 << '\n';
```

This also produces the expected output:

```
1b
```

Using setf() and unsetf() tends to be awkward, so C++ provides a second way to change the formatting options: manipulators. The nice thing about manipulators is that they are smart enough to turn on and off the appropriate flags. Here is an example of using some manipulators to change the base:

```cpp
std::cout << std::hex << 27 << '\n'; // print 27 in hex
std::cout << 28 << '\n'; // we're still in hex
std::cout << std::dec << 29 << '\n'; // back to decimal
```

This program produces the output:

```
1b
1c
29
```

In general, using manipulators is much easier than setting and unsetting flags. Many options are available via both flags and manipulators (such as changing the base), however, other options are only available via flags or via manipulators, so it’s important to know how to use both.

------

#### Useful formatters

Here is a list of some of the more useful flags, manipulators, and member functions. Flags live in the std::ios class, manipulators live in the std namespace, and the member functions live in the std::ostream class.

| Group | Flag                | Meaning                                                      |
| :---- | :------------------ | :----------------------------------------------------------- |
|       | std::ios::boolalpha | If set, booleans print “true” or “false”. If not set, booleans print 0 or 1 |

| Manipulator      | Meaning                          |
| :--------------- | :------------------------------- |
| std::boolalpha   | Booleans print “true” or “false” |
| std::noboolalpha | Booleans print 0 or 1 (default)  |

Example:

```cpp
std::cout << true << ' ' << false << '\n';

std::cout.setf(std::ios::boolalpha);
std::cout << true << ' ' << false << '\n';

std::cout << std::noboolalpha << true << ' ' << false << '\n';

std::cout << std::boolalpha << true << ' ' << false << '\n';
```

Result:

```
1 0
true false
1 0
true false
```

| Group | Flag              | Meaning                                  |
| :---- | :---------------- | :--------------------------------------- |
|       | std::ios::showpos | If set, prefix positive numbers with a + |

| Manipulator    | Meaning                                  |
| :------------- | :--------------------------------------- |
| std::showpos   | Prefixes positive numbers with a +       |
| std::noshowpos | Doesn’t prefix positive numbers with a + |

Example:

```cpp
std::cout << 5 << '\n';

std::cout.setf(std::ios::showpos);
std::cout << 5 << '\n';

std::cout << std::noshowpos << 5 << '\n';

std::cout << std::showpos << 5 << '\n';
```

Result:

```
5
+5
5
+5
```

| Group | Flag                | Meaning                         |
| :---- | :------------------ | :------------------------------ |
|       | std::ios::uppercase | If set, uses upper case letters |

| Manipulator      | Meaning                 |
| :--------------- | :---------------------- |
| std::uppercase   | Uses upper case letters |
| std::nouppercase | Uses lower case letters |

Example:

```cpp
std::cout << 12345678.9 << '\n';

std::cout.setf(std::ios::uppercase);
std::cout << 12345678.9 << '\n';

std::cout << std::nouppercase << 12345678.9 << '\n';

std::cout << std::uppercase << 12345678.9 << '\n';
```

Result:

```
1.23457e+007
1.23457E+007
1.23457e+007
1.23457E+007
```

| Group               | Flag          | Meaning                                                |
| :------------------ | :------------ | :----------------------------------------------------- |
| std::ios::basefield | std::ios::dec | Prints values in decimal (default)                     |
| std::ios::basefield | std::ios::hex | Prints values in hexadecimal                           |
| std::ios::basefield | std::ios::oct | Prints values in octal                                 |
| std::ios::basefield | (none)        | Prints values according to leading characters of value |

| Manipulator | Meaning                      |
| :---------- | :--------------------------- |
| std::dec    | Prints values in decimal     |
| std::hex    | Prints values in hexadecimal |
| std::oct    | Prints values in octal       |

Example:

```cpp
std::cout << 27 << '\n';

std::cout.setf(std::ios::dec, std::ios::basefield);
std::cout << 27 << '\n';

std::cout.setf(std::ios::oct, std::ios::basefield);
std::cout << 27 << '\n';

std::cout.setf(std::ios::hex, std::ios::basefield);
std::cout << 27 << '\n';

std::cout << std::dec << 27 << '\n';
std::cout << std::oct << 27 << '\n';
std::cout << std::hex << 27 << '\n';
```

Result:

```
27
27
33
1b
27
33
1b
```

By now, you should be able to see the relationship between setting formatting via flag and via manipulators. In future examples, we will use manipulators unless they are not available.

------

#### **Precision, notation, and decimal points**

Using manipulators (or flags), it is possible to change the precision and format with which floating point numbers are displayed. There are several formatting options that combine in somewhat complex ways, so we will take a closer look at this.

| Group                | Flag                 | Meaning                                                      |
| :------------------- | :------------------- | :----------------------------------------------------------- |
| std::ios::floatfield | std::ios::fixed      | Uses decimal notation for floating-point numbers             |
| std::ios::floatfield | std::ios::scientific | Uses scientific notation for floating-point numbers          |
| std::ios::floatfield | (none)               | Uses fixed for numbers with few digits, scientific otherwise |
| std::ios::floatfield | std::ios::showpoint  | Always show a decimal point and trailing 0’s for floating-point values |

| Manipulator            | Meaning                                                      |
| :--------------------- | :----------------------------------------------------------- |
| std::fixed             | Use decimal notation for values                              |
| std::scientific        | Use scientific notation for values                           |
| std::showpoint         | Show a decimal point and trailing 0’s for floating-point values |
| std::noshowpoint       | Don’t show a decimal point and trailing 0’s for floating-point values |
| std::setprecision(int) | Sets the precision of floating-point numbers (defined in the iomanip header) |

| Member function               | Meaning                                                      |
| :---------------------------- | :----------------------------------------------------------- |
| std::ios_base::precision()    | Returns the current precision of floating-point numbers      |
| std::ios_base::precision(int) | Sets the precision of floating-point numbers and returns old precision |

If fixed or scientific notation is used, precision determines how many decimal places in the fraction is displayed. Note that if the precision is less than the number of significant digits, the number will be rounded.

```cpp
std::cout << std::fixed << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';

std::cout << std::scientific << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';
```

Produces the result:

```
123.456
123.4560
123.45600
123.456000
123.4560000

1.235e+002
1.2346e+002
1.23456e+002
1.234560e+002
1.2345600e+002
```

If neither fixed nor scientific are being used, precision determines how many significant digits should be displayed. Again, if the precision is less than the number of significant digits, the number will be rounded.

```cpp
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';
```

Produces the following result:

```
123
123.5
123.46
123.456
123.456
```

Using the showpoint manipulator or flag, you can make the stream write a decimal point and trailing zeros.

```cpp
std::cout << std::showpoint << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';
```

Produces the following result:

```
123.
123.5
123.46
123.456
123.4560
```

Here’s a summary table with some more examples:

| Option     | Precision     | 12345.0       | 0.12345    |
| :--------- | :------------ | :------------ | :--------- |
| Normal     | 3             | 1.23e+004     | 0.123      |
| 4          | 1.235e+004    | 0.1235        |            |
| 5          | 12345         | 0.12345       |            |
| 6          | 12345         | 0.12345       |            |
| Showpoint  | 3             | 1.23e+004     | 0.123      |
| 4          | 1.235e+004    | 0.1235        |            |
| 5          | 12345.        | 0.12345       |            |
| 6          | 12345.0       | 0.123450      |            |
| Fixed      | 3             | 12345.000     | 0.123      |
| 4          | 12345.0000    | 0.1235        |            |
| 5          | 12345.00000   | 0.12345       |            |
| 6          | 12345.000000  | 0.123450      |            |
| Scientific | 3             | 1.235e+004    | 1.235e-001 |
| 4          | 1.2345e+004   | 1.2345e-001   |            |
| 5          | 1.23450e+004  | 1.23450e-001  |            |
| 6          | 1.234500e+004 | 1.234500e-001 |            |

------

#### Width, fill characters, and justification

Typically when you print numbers, the numbers are printed without any regard to the space around them. However, it is possible to left or right justify the printing of numbers. In order to do this, we have to first define a field width, which defines the number of output spaces a value will have. If the actual number printed is smaller than the field width, it will be left or right justified (as specified). If the actual number is larger than the field width, it will not be truncated -- it will overflow the field.

| Group                 | Flag               | Meaning                                                      |
| :-------------------- | :----------------- | :----------------------------------------------------------- |
| std::ios::adjustfield | std::ios::internal | Left-justifies the sign of the number, and right-justifies the value |
| std::ios::adjustfield | std::ios::left     | Left-justifies the sign and value                            |
| std::ios::adjustfield | std::ios::right    | Right-justifies the sign and value (default)                 |

| Manipulator        | Meaning                                                      |
| :----------------- | :----------------------------------------------------------- |
| std::internal      | Left-justifies the sign of the number, and right-justifies the value |
| std::left          | Left-justifies the sign and value                            |
| std::right         | Right-justifies the sign and value                           |
| std::setfill(char) | Sets the parameter as the fill character (defined in the iomanip header) |
| std::setw(int)     | Sets the field width for input and output to the parameter (defined in the iomanip header) |

| Member function                | Meaning                                                    |
| :----------------------------- | :--------------------------------------------------------- |
| std::basic_ostream::fill()     | Returns the current fill character                         |
| std::basic_ostream::fill(char) | Sets the fill character and returns the old fill character |
| std::ios_base::width()         | Returns the current field width                            |
| std::ios_base::width(int)      | Sets the current field width and returns old field width   |

In order to use any of these formatters, we first have to set a field width. This can be done via the width(int) member function, or the setw() manipulator. Note that right justification is the default.

```cpp
std::cout << -12345 << '\n'; // print default value with no field width
std::cout << std::setw(10) << -12345 << '\n'; // print default with field width
std::cout << std::setw(10) << std::left << -12345 << '\n'; // print left justified
std::cout << std::setw(10) << std::right << -12345 << '\n'; // print right justified
std::cout << std::setw(10) << std::internal << -12345 << '\n'; // print internally justified
```

This produces the result:

```
-12345
    -12345
-12345
    -12345
-    12345
```

One thing to note is that setw() and width() only affect the next output statement. They are not persistent like some other flags/manipulators.

Now, let’s set a fill character and do the same example:

```cpp
std::cout.fill('*');
std::cout << -12345 << '\n'; // print default value with no field width
std::cout << std::setw(10) << -12345 << '\n'; // print default with field width
std::cout << std::setw(10) << std::left << -12345 << '\n'; // print left justified
std::cout << std::setw(10) << std::right << -12345 << '\n'; // print right justified
std::cout << std::setw(10) << std::internal << -12345 << '\n'; // print internally justified
```

This produces the output:

```
-12345
****-12345
-12345****
****-12345
-****12345
```

Note that all the blank spaces in the field have been filled up with the fill character.

The ostream class and iostream library contain other output functions, flags, and manipulators that may be useful, depending on what you need to do. As with the istream class, those topics are really more suited for a tutorial or book focusing on the standard library (such as the excellent book “The C++ Standard Template Library” by Nicolai M. Josuttis).

------

### 23.4 — Stream classes for strings

So far, all of the I/O examples you have seen have been writing to cout or reading from cin. However, there is another set of classes called the stream classes for strings that allow you to use the familiar insertions (<<) and extraction (>>) operators to work with strings. Like istream and ostream, the string streams provide a buffer to hold data. However, unlike cin and cout, these streams are not connected to an I/O channel (such as a keyboard, monitor, etc…). One of the primary uses of string streams is to buffer output for display at a later time, or to process input line-by-line.

There are six stream classes for strings: istringstream (derived from istream), ostringstream (derived from ostream), and stringstream (derived from iostream) are used for reading and writing normal characters width strings. wistringstream, wostringstream, and wstringstream are used for reading and writing wide character strings. To use the stringstreams, you need to #include the sstream header.

There are two ways to get data into a stringstream:

1. Use the insertion (<<) operator:

   ```c++
   std::stringstream os;
   os << "en garde!\n"; // insert "en garde!" into the stringstream
   ```

2. Use the str(string) function to set the value of the buffer:

   ```c++
   std::stringstream os;
   os.str("en garde!"); // set the stringstream buffer to "en garde!"
   ```

There are similarly two ways to get data out of a stringstream:

1. Use the str() function to retrieve the results of the buffer:

   ```c++
   std::stringstream os;
   os << "12345 67.89\n";
   std::cout << os.str();
   ```

   This prints:

   ```
   12345 67.89
   ```

2. Use the extraction (>>) operator:

   ```
   std::stringstream os;
   os << "12345 67.89"; // insert a string of numbers into the stream
   
   std::string strValue;
   os >> strValue;
   
   std::string strValue2;
   os >> strValue2;
   
   // print the numbers separated by a dash
   std::cout << strValue << " - " << strValue2 << '\n';
   ```

   This program prints:

   ```
   12345 - 67.89
   ```

Note that the >> operator iterates through the string -- each successive use of >> returns the next extractable value in the stream. On the other hand, str() returns the whole value of the stream, even if the >> has already been used on the stream.

------

#### **Conversion between strings and numbers**

Because the insertion and extraction operators know how to work with all of the basic data types, we can use them in order to convert strings to numbers or vice versa.

First, let’s take a look at converting numbers into a string:

```cpp
std::stringstream os;

int nValue{ 12345 };
double dValue{ 67.89 };
os << nValue << ' ' << dValue;

std::string strValue1, strValue2;
os >> strValue1 >> strValue2;

std::cout << strValue1 << ' ' << strValue2 << '\n';
```

This snippet prints:

```
12345 67.89
```

Now let’s convert a numerical string to a number:

```cpp
std::stringstream os;
os << "12345 67.89"; // insert a string of numbers into the stream
int nValue;
double dValue;

os >> nValue >> dValue;

std::cout << nValue << ' ' << dValue << '\n';
```

This program prints:

```
12345 67.89
```

------

#### **Clearing a stringstream for reuse**

There are several ways to empty a stringstream’s buffer.

1. Set it to the empty string using str() with a blank C-style string:

   ```
   std::stringstream os;
   os << "Hello ";
   
   os.str(""); // erase the buffer
   
   os << "World!";
   std::cout << os.str();
   ```

2. Set it to the empty string using str() with a blank std::string object:

   ```
   std::stringstream os;
   os << "Hello ";
   
   os.str(std::string{}); // erase the buffer
   
   os << "World!";
   std::cout << os.str();
   ```

Both of these programs produce the following result:

```
World!
```

When clearing out a stringstream, it is also generally a good idea to call the clear() function:

```cpp
std::stringstream os;
os << "Hello ";

os.str(""); // erase the buffer
os.clear(); // reset error flags

os << "World!";
std::cout << os.str();
```

clear() resets any error flags that may have been set and returns the stream back to the ok state. We will talk more about the stream state and error flags in the next lesson.

------

### 23.5 — Stream states and input validation

#### Stream states

The ios_base class contains several state flags that are used to signal various conditions that may occur when using streams:

| Flag    | Meaning                                                      |
| :------ | :----------------------------------------------------------- |
| goodbit | Everything is okay                                           |
| badbit  | Some kind of fatal error occurred (e.g. the program tried to read past the end of a file) |
| eofbit  | The stream has reached the end of a file                     |
| failbit | A non-fatal error occurred (e.g. the user entered letters when the program was expecting an integer) |

Although these flags live in ios_base, because ios is derived from ios_base and ios takes less typing than ios_base, they are generally accessed through ios (e.g. as std::ios::failbit).

ios also provides a number of member functions in order to conveniently access these states:

| Member function | Meaning                                                      |
| :-------------- | :----------------------------------------------------------- |
| good()          | Returns true if the goodbit is set (the stream is ok)        |
| bad()           | Returns true if the badbit is set (a fatal error occurred)   |
| eof()           | Returns true if the eofbit is set (the stream is at the end of a file) |
| fail()          | Returns true if the failbit is set (a non-fatal error occurred) |
| clear()         | Clears all flags and restores the stream to the goodbit state |
| clear(state)    | Clears all flags and sets the state flag passed in           |
| rdstate()       | Returns the currently set flags                              |
| setstate(state) | Sets the state flag passed in                                |

The most commonly dealt with bit is the failbit, which is set when the user enters invalid input. For example, consider the following program:

```cpp
std::cout << "Enter your age: ";
int age {};
std::cin >> age;
```

Note that this program is expecting the user to enter an integer. However, if the user enters non-numeric data, such as “Alex”, cin will be unable to extract anything to age, and the failbit will be set.

If an error occurs and a stream is set to anything other than goodbit, further stream operations on that stream will be ignored. This condition can be cleared by calling the clear() function.

------

#### Input validation

**Input validation** is the process of checking whether the user input meets some set of criteria. Input validation can generally be broken down into two types: string and numeric.

With string validation, we accept all user input as a string, and then accept or reject that string depending on whether it is formatted appropriately. For example, if we ask the user to enter a telephone number, we may want to ensure the data they enter has ten digits. In most languages (especially scripting languages like Perl and PHP), this is done via regular expressions. The C++ standard library has a [regular expression library](https://en.cppreference.com/w/cpp/regex) as well. Because regular expressions are slow compared to manual string validation, they should only be used if performance (compile-time and run-time) is of no concern or manual validation is too cumbersome.

With numerical validation, we are typically concerned with making sure the number the user enters is within a particular range (e.g. between 0 and 20). However, unlike with string validation, it’s possible for the user to enter things that aren’t numbers at all -- and we need to handle these cases too.

To help us out, C++ provides a number of useful functions that we can use to determine whether specific characters are numbers or letters. The following functions live in the cctype header:

| Function           | Meaning                                                      |
| :----------------- | :----------------------------------------------------------- |
| std::isalnum(int)  | Returns non-zero if the parameter is a letter or a digit     |
| std::isalpha(int)  | Returns non-zero if the parameter is a letter                |
| std::iscntrl(int)  | Returns non-zero if the parameter is a control character     |
| std::isdigit(int)  | Returns non-zero if the parameter is a digit                 |
| std::isgraph(int)  | Returns non-zero if the parameter is printable character that is not whitespace |
| std::isprint(int)  | Returns non-zero if the parameter is printable character (including whitespace) |
| std::ispunct(int)  | Returns non-zero if the parameter is neither alphanumeric nor whitespace |
| std::isspace(int)  | Returns non-zero if the parameter is whitespace              |
| std::isxdigit(int) | Returns non-zero if the parameter is a hexadecimal digit (0-9, a-f, A-F) |

------

#### String validation

Let’s do a simple case of string validation by asking the user to enter their name. Our validation criteria will be that the user enters only alphabetic characters or spaces. If anything else is encountered, the input will be rejected.

When it comes to variable length inputs, the best way to validate strings (besides using a regular expression library) is to step through each character of the string and ensure it meets the validation criteria. That’s exactly what we’ll do here, or better, that’s what `std::all_of` does for us.

```cpp
#include <algorithm> // std::all_of
#include <cctype> // std::isalpha, std::isspace
#include <iostream>
#include <ranges>
#include <string>
#include <string_view>

bool isValidName(std::string_view name)
{
  return std::ranges::all_of(name, [](char ch) {
    return (std::isalpha(ch) || std::isspace(ch));
  });

  // Before C++20, without ranges
  // return std::all_of(name.begin(), name.end(), [](char ch) {
  //    return (std::isalpha(ch) || std::isspace(ch));
  // });
}

int main()
{
  std::string name{};

  do
  {
    std::cout << "Enter your name: ";
    std::getline(std::cin, name); // get the entire line, including spaces
  } while (!isValidName(name));

  std::cout << "Hello " << name << "!\n";
}
```

Note that this code isn’t perfect: the user could say their name was “asf w jweo s di we ao” or some other bit of gibberish, or even worse, just a bunch of spaces. We could address this somewhat by refining our validation criteria to only accept strings that contain at least one character and at most one space.

**Author’s note**

Reader “Waldo” provides a C++20 solution (using std::ranges) that addresses these shortcomings [here](https://www.learncpp.com/cpp-tutorial/stream-states-and-input-validation/#comment-571052)

Now let’s take a look at another example where we are going to ask the user to enter their phone number. Unlike a user’s name, which is variable-length and where the validation criteria are the same for every character, a phone number is a fixed length but the validation criteria differ depending on the position of the character. Consequently, we are going to take a different approach to validating our phone number input. In this case, we’re going to write a function that will check the user’s input against a predetermined template to see whether it matches. The template will work as follows:

A # will match any digit in the user input.
A @ will match any alphabetic character in the user input.
A _ will match any whitespace.
A ? will match anything.
Otherwise, the characters in the user input and the template must match exactly.

So, if we ask the function to match the template “(###) ###-####”, that means we expect the user to enter a ‘(‘ character, three numbers, a ‘)’ character, a space, three numbers, a dash, and four more numbers. If any of these things doesn’t match, the input will be rejected.

Here is the code:

```cpp
#include <algorithm> // std::equal
#include <cctype> // std::isdigit, std::isspace, std::isalpha
#include <iostream>
#include <map>
#include <ranges>
#include <string>
#include <string_view>

bool inputMatches(std::string_view input, std::string_view pattern)
{
    if (input.length() != pattern.length())
    {
        return false;
    }

    // This table defines all special symbols that can match a range of user input
    // Each symbol is mapped to a function that determines whether the input is valid for that symbol
    static const std::map<char, int (*)(int)> validators{
      { '#', &std::isdigit },
      { '_', &std::isspace },
      { '@', &std::isalpha },
      { '?', [](int) { return 1; } }
    };

    // Before C++20, use
    // return std::equal(input.begin(), input.end(), pattern.begin(), [](char ch, char mask) -> bool {
    // ...

    return std::ranges::equal(input, pattern, [](char ch, char mask) -> bool {
        if (auto found{ validators.find(mask) }; found != validators.end())
        {
            // The pattern's current element was found in the validators. Call the
            // corresponding function.
            return (*found->second)(ch);
        }
        else
        {
            // The pattern's current element was not found in the validators. The
            // characters have to be an exact match.
            return (ch == mask);
        }
        });
}

int main()
{
    std::string phoneNumber{};

    do
    {
        std::cout << "Enter a phone number (###) ###-####: ";
        std::getline(std::cin, phoneNumber);
    } while (!inputMatches(phoneNumber, "(###) ###-####"));

    std::cout << "You entered: " << phoneNumber << '\n';
}
```

Using this function, we can force the user to match our specific format exactly. However, this function is still subject to several constraints: if #, @, _, and ? are valid characters in the user input, this function won’t work, because those symbols have been given special meanings. Also, unlike with regular expressions, there is no template symbol that means “a variable number of characters can be entered”. Thus, such a template could not be used to ensure the user enters two words separated by a whitespace, because it can not handle the fact that the words are of variable lengths. For such problems, the non-template approach is generally more appropriate.

------

#### Numeric validation

When dealing with numeric input, the obvious way to proceed is to use the extraction operator to extract input to a numeric type. By checking the failbit, we can then tell whether the user entered a number or not.

Let’s try this approach:

```cpp
#include <iostream>
#include <limits>

int main()
{
    int age{};

    while (true)
    {
        std::cout << "Enter your age: ";
        std::cin >> age;

        if (std::cin.fail()) // no extraction took place
        {
            std::cin.clear(); // reset the state bits back to goodbit so we can use ignore()
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // clear out the bad input from the stream
            continue; // try again
        }

        if (age <= 0) // make sure age is positive
            continue;

        break;
    }

    std::cout << "You entered: " << age << '\n';
}
```

If the user enters an integer, the extraction will succeed. std::cin.fail() will evaluate to false, skipping the conditional, and (assuming the user entered a positive number), we will hit the break statement, exiting the loop.

If the user instead enters input starting with a letter, the extraction will fail. std::cin.fail() will evaluate to true, and we will go into the conditional. At the end of the conditional block, we will hit the continue statement, which will jump back to the top of the while loop, and the user will be asked to enter input again.

However, there’s one more case we haven’t tested for, and that’s when the user enters a string that starts with numbers but then contains letters (e.g. “34abcd56”). In this case, the starting numbers (34) will be extracted into age, the remainder of the string (“abcd56”) will be left in the input stream, and the failbit will NOT be set. This causes two potential problems:

1. If you want this to be valid input, you now have garbage in your stream.
2. If you don’t want this to be valid input, it is not rejected (and you have garbage in your stream).

Let’s fix the first problem. This is easy:

```cpp
#include <iostream>
#include <limits>

int main()
{
    int age{};

    while (true)
    {
        std::cout << "Enter your age: ";
        std::cin >> age;

        if (std::cin.fail()) // no extraction took place
        {
            std::cin.clear(); // reset the state bits back to goodbit so we can use ignore()
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // clear out the bad input from the stream
            continue; // try again
        }

        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // clear out any additional input from the stream

        if (age <= 0) // make sure age is positive
            continue;

      break;
    }

    std::cout << "You entered: " << age << '\n';
}
```

If you don’t want such input to be valid, we’ll have to do a little extra work. Fortunately, the previous solution gets us half way there. We can use the gcount() function to determine how many characters were ignored. If our input was valid, gcount() should return 1 (the newline character that was discarded). If it returns more than 1, the user entered something that wasn’t extracted properly, and we should ask them for new input. Here’s an example of this:

```cpp
#include <iostream>
#include <limits>

int main()
{
    int age{};

    while (true)
    {
        std::cout << "Enter your age: ";
        std::cin >> age;

        if (std::cin.fail()) // no extraction took place
        {
            std::cin.clear(); // reset the state bits back to goodbit so we can use ignore()
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // clear out the bad input from the stream
            continue; // try again
        }

        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // clear out any additional input from the stream
        if (std::cin.gcount() > 1) // if we cleared out more than one additional character
        {
            continue; // we'll consider this input to be invalid
        }

        if (age <= 0) // make sure age is positive
        {
            continue;
        }

        break;
    }

    std::cout << "You entered: " << age << '\n';
}
```

------

#### Numeric validation as a string

The above example was quite a bit of work simply to get a simple value! Another way to process numeric input is to read it in as a string, then try to convert it to a numeric type. The following program makes use of that methodology:

```cpp
#include <charconv> // std::from_chars
#include <iostream>
#include <optional>
#include <string>
#include <string_view>

std::optional<int> extractAge(std::string_view age)
{
  int result{};
  auto end{ age.data() + age.length() };

  // Try to parse an int from age
  if (std::from_chars(age.data(), end, result).ptr != end)
  {
    return {};
  }

  if (result <= 0) // make sure age is positive
  {
    return {};
  }

  return result;
}

int main()
{
  int age{};

  while (true)
  {
    std::cout << "Enter your age: ";
    std::string strAge{};
    std::getline(std::cin >> std::ws, strAge);

    if (auto extracted{ extractAge(strAge) })
    {
      age = *extracted;
      break;
    }
  }

  std::cout << "You entered: " << age << '\n';
}
```

Whether this approach is more or less work than straight numeric extraction depends on your validation parameters and restrictions.

As you can see, doing input validation in C++ is a lot of work. Fortunately, many such tasks (e.g. doing numeric validation as a string) can be easily turned into functions that can be reused in a wide variety of situations.

------

### 23.6 — Basic file I/O

**Author’s note**

This lesson and the next were written pre-C++17. In C++17, file I/O is better done using `std::filesystem`.

File I/O in C++ works very similarly to normal I/O (with a few minor added complexities). There are 3 basic file I/O classes in C++: ifstream (derived from istream), ofstream (derived from ostream), and fstream (derived from iostream). These classes do file input, output, and input/output respectively. To use the file I/O classes, you will need to include the fstream header.

Unlike the cout, cin, cerr, and clog streams, which are already ready for use, file streams have to be explicitly set up by the programmer. However, this is extremely simple: to open a file for reading and/or writing, simply instantiate an object of the appropriate file I/O class, with the name of the file as a parameter. Then use the insertion (<<) or extraction (>>) operator to write to or read data from the file. Once you are done, there are several ways to close a file: explicitly call the close() function, or just let the file I/O variable go out of scope (the file I/O class destructor will close the file for you).

------

#### File output

To do file output in the following example, we’re going to use the ofstream class. This is extremely straighforward:

```cpp
#include <fstream>
#include <iostream>

int main()
{
    // ofstream is used for writing files
    // We'll make a file called Sample.txt
    std::ofstream outf{ "Sample.txt" };

    // If we couldn't open the output file stream for writing
    if (!outf)
    {
        // Print an error and exit
        std::cerr << "Uh oh, Sample.txt could not be opened for writing!\n";
        return 1;
    }

    // We'll write two lines into this file
    outf << "This is line 1\n";
    outf << "This is line 2\n";

    return 0;

    // When outf goes out of scope, the ofstream
    // destructor will close the file
}
```

If you look in your project directory, you should see a file called Sample.txt. If you open it with a text editor, you will see that it indeed contains two lines we wrote to the file.

Note that it is also possible to use the put() function to write a single character to the file.

------

#### File input

Now, we’ll take the file we wrote in the last example and read it back in from disk. Note that ifstream returns a 0 if we’ve reached the end of the file (EOF). We’ll use this fact to determine how much to read.

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    // ifstream is used for reading files
    // We'll read from a file called Sample.txt
    std::ifstream inf{ "Sample.txt" };

    // If we couldn't open the output file stream for reading
    if (!inf)
    {
        // Print an error and exit
        std::cerr << "Uh oh, Sample.txt could not be opened for reading!\n";
        return 1;
    }

    // While there's still stuff left to read
    while (inf)
    {
        // read stuff from the file into a string and print it
        std::string strInput;
        inf >> strInput;
        std::cout << strInput << '\n';
    }

    return 0;

    // When inf goes out of scope, the ifstream
    // destructor will close the file
}
```

This produces the result:

```
This
is
line
1
This
is
line
2
```

Hmmm, that wasn’t quite what we wanted. Remember that the extraction operator breaks on whitespace. In order to read in entire lines, we’ll have to use the getline() function.

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    // ifstream is used for reading files
    // We'll read from a file called Sample.txt
    std::ifstream inf{ "Sample.txt" };

    // If we couldn't open the input file stream for reading
    if (!inf)
    {
        // Print an error and exit
        std::cerr << "Uh oh, Sample.txt could not be opened for reading!\n";
        return 1;
    }

    // While there's still stuff left to read
    while (inf)
    {
        // read stuff from the file into a string and print it
        std::string strInput;
        std::getline(inf, strInput);
        std::cout << strInput << '\n';
    }

    return 0;

    // When inf goes out of scope, the ifstream
    // destructor will close the file
}
```

This produces the result:

```
This is line 1
This is line 2
```

------

#### Buffered output

Output in C++ may be buffered. This means that anything that is output to a file stream may not be written to disk immediately. Instead, several output operations may be batched and handled together. This is done primarily for performance reasons. When a buffer is written to disk, this is called **flushing** the buffer. One way to cause the buffer to be flushed is to close the file -- the contents of the buffer will be flushed to disk, and then the file will be closed.

Buffering is usually not a problem, but in certain circumstance it can cause complications for the unwary. The main culprit in this case is when there is data in the buffer, and then program terminates immediately (either by crashing, or by calling exit()). In these cases, the destructors for the file stream classes are not executed, which means the files are never closed, which means the buffers are never flushed. In this case, the data in the buffer is not written to disk, and is lost forever. This is why it is always a good idea to explicitly close any open files before calling exit().

It is possible to flush the buffer manually using the ostream::flush() function or sending std::flush to the output stream. Either of these methods can be useful to ensure the contents of the buffer are written to disk immediately, just in case the program crashes.

One interesting note is that std::endl; also flushes the output stream. Consequently, overuse of std::endl (causing unnecessary buffer flushes) can have performance impacts when doing buffered I/O where flushes are expensive (such as writing to a file). For this reason, performance conscious programmers will often use ‘\n’ instead of std::endl to insert a newline into the output stream, to avoid unnecessary flushing of the buffer.

------

#### File modes

What happens if we try to write to a file that already exists? Running the output example again shows that the original file is completely overwritten each time the program is run. What if, instead, we wanted to append some more data to the end of the file? It turns out that the file stream constructors take an optional second parameter that allows you to specify information about how the file should be opened. This parameter is called mode, and the valid flags that it accepts live in the ios class.

| Ios file mode | Meaning                                              |
| :------------ | :--------------------------------------------------- |
| app           | Opens the file in append mode                        |
| ate           | Seeks to the end of the file before reading/writing  |
| binary        | Opens the file in binary mode (instead of text mode) |
| in            | Opens the file in read mode (default for ifstream)   |
| out           | Opens the file in write mode (default for ofstream)  |
| trunc         | Erases the file if it already exists                 |

It is possible to specify multiple flags by bitwise ORing them together (using the | operator). ifstream defaults to std::ios::in file mode. ofstream defaults to std::ios::out file mode. And fstream defaults to std::ios::in | std::ios::out file mode, meaning you can both read and write by default.

**Tip**

Due to the way fstream was designed, it may fail if std::ios::in is used and the file being opened does not exist. If you need to create a new file using fstream, use std::ios::out mode only.

Let’s write a program that appends two more lines to the Sample.txt file we previously created:

```cpp
#include <iostream>
#include <fstream>

int main()
{
    // We'll pass the ios:app flag to tell the ofstream to append
    // rather than rewrite the file. We do not need to pass in std::ios::out
    // because ofstream defaults to std::ios::out
    std::ofstream outf{ "Sample.txt", std::ios::app };

    // If we couldn't open the output file stream for writing
    if (!outf)
    {
        // Print an error and exit
        std::cerr << "Uh oh, Sample.txt could not be opened for writing!\n";
        return 1;
    }

    outf << "This is line 3\n";
    outf << "This is line 4\n";

    return 0;

    // When outf goes out of scope, the ofstream
    // destructor will close the file
}
```

Now if we take a look at Sample.txt (using one of the above sample programs that prints its contents, or loading it in a text editor), we will see the following:

```
This is line 1
This is line 2
This is line 3
This is line 4
```

------

#### Explicitly opening files using open()

Just like it is possible to explicitly close a file stream using close(), it’s also possible to explicitly open a file stream using open(). open() works just like the file stream constructors -- it takes a file name and an optional file mode.

For example:

```cpp
std::ofstream outf{ "Sample.txt" };
outf << "This is line 1\n";
outf << "This is line 2\n";
outf.close(); // explicitly close the file

// Oops, we forgot something
outf.open("Sample.txt", std::ios::app);
outf << "This is line 3\n";
outf.close();
```

You can find more information about the open() function [here](https://en.cppreference.com/w/cpp/io/basic_filebuf/open).

------

### 23.7 — Random file I/O

#### The file pointer

Each file stream class contains a file pointer that is used to keep track of the current read/write position within the file. When something is read from or written to a file, the reading/writing happens at the file pointer’s current location. By default, when opening a file for reading or writing, the file pointer is set to the beginning of the file. However, if a file is opened in append mode, the file pointer is moved to the end of the file, so that writing does not overwrite any of the current contents of the file.

------

#### Random file access with seekg() and seekp()

So far, all of the file access we’ve done has been sequential -- that is, we’ve read or written the file contents in order. However, it is also possible to do random file access -- that is, skip around to various points in the file to read its contents. This can be useful when your file is full of records, and you wish to retrieve a specific record. Rather than reading all of the records until you get to the one you want, you can skip directly to the record you wish to retrieve.

Random file access is done by manipulating the file pointer using either seekg() function (for input) and seekp() function (for output). In case you are wondering, the g stands for “get” and the p for “put”. For some types of streams, seekg() (changing the read position) and seekp() (changing the write position) operate independently -- however, with file streams, the read and write position are always identical, so seekg and seekp can be used interchangeably.

The seekg() and seekp() functions take two parameters. The first parameter is an offset that determines how many bytes to move the file pointer. The second parameter is an ios flag that specifies what the offset parameter should be offset from.

| Ios seek flag | Meaning                                                      |
| :------------ | :----------------------------------------------------------- |
| beg           | The offset is relative to the beginning of the file (default) |
| cur           | The offset is relative to the current location of the file pointer |
| end           | The offset is relative to the end of the file                |

A positive offset means move the file pointer towards the end of the file, whereas a negative offset means move the file pointer towards the beginning of the file.

Here are some examples:

```cpp
inf.seekg(14, std::ios::cur); // move forward 14 bytes
inf.seekg(-18, std::ios::cur); // move backwards 18 bytes
inf.seekg(22, std::ios::beg); // move to 22nd byte in file
inf.seekg(24); // move to 24th byte in file
inf.seekg(-28, std::ios::end); // move to the 28th byte before end of the file
```

Moving to the beginning or end of the file is easy:

```cpp
inf.seekg(0, std::ios::beg); // move to beginning of file
inf.seekg(0, std::ios::end); // move to end of file
```

Let’s do an example using seekg() and the input file we created in the last lesson. That input file looks like this:

```
This is line 1
This is line 2
This is line 3
This is line 4
```

Here is the example:

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    std::ifstream inf{ "Sample.txt" };

    // If we couldn't open the input file stream for reading
    if (!inf)
    {
        // Print an error and exit
        std::cerr << "Uh oh, Sample.txt could not be opened for reading!\n";
        return 1;
    }

    std::string strData;

    inf.seekg(5); // move to 5th character
    // Get the rest of the line and print it, moving to line 2
    std::getline(inf, strData);
    std::cout << strData << '\n';

    inf.seekg(8, std::ios::cur); // move 8 more bytes into file
    // Get rest of the line and print it
    std::getline(inf, strData);
    std::cout << strData << '\n';

    inf.seekg(-14, std::ios::end); // move 14 bytes before end of file
    // Get rest of the line and print it
    std::getline(inf, strData);
    std::cout << strData << '\n';

    return 0;
}
```

This produces the result:

```
is line 1
line 2
This is line 4
```

Note: Some compilers have buggy implementations of seekg() and seekp() when used in conjunction with text files (due to buffering). If your compiler is one of them (and you’ll know because your output will differ from the above), you can try opening the file in binary mode instead:

```cpp
std::ifstream inf {"Sample.txt", std::ifstream::binary};
```

Two other useful functions are tellg() and tellp(), which return the absolute position of the file pointer. This can be used to determine the size of a file:

```cpp
std::ifstream inf {"Sample.txt"};
inf.seekg(0, std::ios::end); // move to end of file
std::cout << inf.tellg();
```

On the author’s machine, this prints:

```
64
```

which is how long sample.txt is in bytes (assuming a newline after the last line).

**Author’s note**

In programming, a newline (‘\n’) is actually an abstraction.

On Windows, a newline is represented as sequential CR (carriage return) and LF (line feed) characters (thus taking 2 bytes of storage).
On Unix, a newline is represented as a LF (line feed) character (thus taking 1 byte of storage).

The result of `64` in the prior example occurred on Windows. If you run the example on Unix, you’ll get `60` instead, due to the smaller newline representation.

------

#### Reading and writing a file at the same time using fstream

The fstream class is capable of both reading and writing a file at the same time -- almost! The big caveat here is that it is not possible to switch between reading and writing arbitrarily. Once a read or write has taken place, the only way to switch between the two is to perform an operation that modifies the file position (e.g. a seek). If you don’t actually want to move the file pointer (because it’s already in the spot you want), you can always seek to the current position:

```cpp
// assume iofile is an object of type fstream
iofile.seekg(iofile.tellg(), std::ios::beg); // seek to current file position
```

If you do not do this, any number of strange and bizarre things may occur.

(Note: Although it may seem that `iofile.seekg(0, std::ios::cur)` would also work, it appears some compilers may optimize this away).

One other bit of trickiness: Unlike ifstream, where we could say `while (inf)` to determine if there was more to read, this will not work with fstream.

Let’s do a file I/O example using fstream. We’re going to write a program that opens a file, reads its contents, and changes any vowels it finds to a ‘#’ symbol.

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    // Note we have to specify both in and out because we're using fstream
    std::fstream iofile{ "Sample.txt", std::ios::in | std::ios::out };

    // If we couldn't open iofile, print an error
    if (!iofile)
    {
        // Print an error and exit
        std::cerr << "Uh oh, Sample.txt could not be opened!\n";
        return 1;
    }

    char chChar{}; // we're going to do this character by character

    // While there's still data to process
    while (iofile.get(chChar))
    {
        switch (chChar)
        {
            // If we find a vowel
            case 'a':
            case 'e':
            case 'i':
            case 'o':
            case 'u':
            case 'A':
            case 'E':
            case 'I':
            case 'O':
            case 'U':

                // Back up one character
                iofile.seekg(-1, std::ios::cur);

                // Because we did a seek, we can now safely do a write, so
                // let's write a # over the vowel
                iofile << '#';

                // Now we want to go back to read mode so the next call
                // to get() will perform correctly.  We'll seekg() to the current
                // location because we don't want to move the file pointer.
                iofile.seekg(iofile.tellg(), std::ios::beg);

                break;
        }
    }

    return 0;
}
```

After running the above program, our Sample.txt file will look like this:

```
Th#s #s l#n# 1
Th#s #s l#n# 2
Th#s #s l#n# 3
Th#s #s l#n# 4
```

------

#### Other useful file functions

To delete a file, simply use the remove() function.

Also, the is_open() function will return true if the stream is currently open, and false otherwise.

------

#### A warning about writing pointers to disk

While streaming variables to a file is quite easy, things become more complicated when you’re dealing with pointers. Remember that a pointer simply holds the address of the variable it is pointing to. Although it is possible to read and write addresses to disk, it is extremely dangerous to do so. This is because a variable’s address may differ from execution to execution. Consequently, although a variable may have lived at address 0x0012FF7C when you wrote that address to disk, it may not live there any more when you read that address back in!

For example, let’s say you had an integer named nValue that lived at address 0x0012FF7C. You assigned nValue the value 5. You also declared a ointer named *pnValue that points to nValue. pnValue holds nValue’s address of 0x0012FF7C. You want to save these for later, so you write the value 5 and the address 0x0012FF7C to disk.

A few weeks later, you run the program again and read these values back from disk. You read the value 5 into another variable named nValue, which lives at 0x0012FF78. You read the address 0x0012FF7C into a new pointer named *pnValue. Because pnValue now points to 0x0012FF7C when the nValue lives at 0x0012FF78, pnValue is no longer pointing to nValue, and trying to access pnValue will lead you into trouble.

**Warning**

Do not write memory addresses to files. The variables that were originally at those addresses may be at different addresses when you read their values back in from disk, and the addresses will be invalid.