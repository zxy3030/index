## **std::string**

### 22.1 — std::string and std::wstring

The standard library contains many useful classes -- but perhaps the most useful is std::string. std::string (and std::wstring) is a string class that provides many operations to assign, compare, and modify strings. In this chapter, we’ll look into these string classes in depth.

Note: C-style strings will be referred to as “C-style strings”, whereas std::string (and std::wstring) will be referred to simply as “strings”.

**Author’s note**

This chapter is somewhat outdated and will likely be condensed in a future update. Feel free to scan the material for ideas and useful examples, but technical reference sites (e.g. [cppreference](https://en.cppreference.com/w/cpp/string/basic_string)) should be preferred for the most up-to-date information.

------

#### **Motivation for a string class**

In a previous lesson, we covered [C-style strings](https://www.learncpp.com/cpp-tutorial/66-c-style-strings/), which uses char arrays to store a string of characters. If you’ve tried to do anything with C-style strings, you’ll very quickly come to the conclusion that they are a pain to work with, easy to mess up, and hard to debug.

C-style strings have many shortcomings, primarily revolving around the fact that you have to do all the memory management yourself. For example, if you want to assign the string “hello!” into a buffer, you have to first dynamically allocate a buffer of the correct length:

```cpp
char* strHello { new char[7] };
```

Don’t forget to account for an extra character for the null terminator!

Then you have to actually copy the value in:

```cpp
strcpy(strHello, "hello!");
```

Hopefully you made your buffer large enough so there’s no buffer overflow!

And of course, because the string is dynamically allocated, you have to remember to deallocate it properly when you’re done with it:

```cpp
delete[] strHello;
```

Don’t forget to use array delete instead of normal delete!

Furthermore, many of the intuitive operators that C provides to work with numbers, such as assignment and comparisons, simply don’t work with C-style strings. Sometimes these will appear to work but actually produce incorrect results -- for example, comparing two C-style strings using == will actually do a pointer comparison, not a string comparison. Assigning one C-style string to another using operator= will appear to work at first, but is actually doing a pointer copy (shallow copy), which is not generally what you want. These kinds of things can lead to program crashes that are very hard to find and debug!

The bottom line is that working with C-style strings requires remembering a lot of nit-picky rules about what is safe/unsafe, memorizing a bunch of functions that have funny names like strcat() and strcmp() instead of using intuitive operators, and doing lots of manual memory management.

Fortunately, C++ and the standard library provide a much better way to deal with strings: the std::string and std::wstring classes. By making use of C++ concepts such as constructors, destructors, and operator overloading, std::string allows you to create and manipulate strings in an intuitive and safe manner! No more memory management, no more weird function names, and a much reduced potential for disaster.

Sign me up!

------

#### **String overview**

All string functionality in the standard library lives in the header file. To use it, simply include the string header:

```cpp
#include <string>
```

There are actually 3 different string classes in the string header. The first is a templated base class named basic_string<>:

```cpp
namespace std
{
    template<class charT, class traits = char_traits<charT>, class Allocator = allocator<charT> >
        class basic_string;
}
```

You won’t be working with this class directly, so don’t worry about what traits or an Allocator is for the time being. The default values will suffice in almost every imaginable case.

There are two flavors of basic_string<> provided by the standard library:

```cpp
namespace std
{
    typedef basic_string<char> string;
    typedef basic_string<wchar_t> wstring;
}
```

These are the two classes that you will actually use. std::string is used for standard ascii and utf-8 strings. std::wstring is used for wide-character/unicode (utf-16) strings. There is no built-in class for utf-32 strings (though you should be able to extend your own from basic_string<> if you need one).

Although you will directly use std::string and std::wstring, all of the string functionality is implemented in the basic_string<> class. String and wstring are able to access that functionality directly by virtue of being templated. Consequently, all of the functions presented will work for both string and wstring. However, because basic_string is a templated class, it also means the compiler will produce horrible looking template errors when you do something syntactically incorrect with a string or wstring. Don’t be intimidated by these errors; they look far worse than they are!

Here’s a list of all the functions in the string class. Most of these functions have multiple flavors to handle different types of inputs, which we will cover in more depth in the next lessons.

| Function                                                     | Effect                                                       |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Creation and destruction**                                 |                                                              |
| [(constructor)](https://www.learncpp.com/cpp-tutorial/17-2-ststring-construction-and-destruction/) [(destructor)](https://www.learncpp.com/cpp-tutorial/17-2-ststring-construction-and-destruction/) | Create or copy a string Destroy a string                     |
| **Size and capacity**                                        |                                                              |
| [capacity()](https://www.learncpp.com/cpp-tutorial/17-3-stdstring-length-and-capacity/) [empty()](https://www.learncpp.com/cpp-tutorial/17-3-stdstring-length-and-capacity/) [length(), size()](https://www.learncpp.com/cpp-tutorial/17-3-stdstring-length-and-capacity/) [max_size()](https://www.learncpp.com/cpp-tutorial/17-3-stdstring-length-and-capacity/) [reserve()](https://www.learncpp.com/cpp-tutorial/17-3-stdstring-length-and-capacity/) | Returns the number of characters that can be held without reallocation Returns a boolean indicating whether the string is empty Returns the number of characters in string Returns the maximum string size that can be allocated Expand or shrink the capacity of the string |
| **Element access**                                           |                                                              |
| [[\], at()](https://www.learncpp.com/cpp-tutorial/17-4-stdstring-character-access-and-conversion-to-c-style-arrays/) | Accesses the character at a particular index                 |
| **Modification**                                             |                                                              |
| [=, assign()](https://www.learncpp.com/cpp-programming/17-5-stdstring-assignment-and-swapping/) [+=, append(), push_back()](https://www.learncpp.com/uncategorized/17-6-stdstring-appending/) [insert()](https://www.learncpp.com/cpp-tutorial/17-7-stdstring-inserting/) clear() erase() replace() resize() [swap()](https://www.learncpp.com/cpp-programming/17-5-stdstring-assignment-and-swapping/) | Assigns a new value to the string Concatenates characters to end of the string Inserts characters at an arbitrary index in string Delete all characters in the string Erase characters at an arbitrary index in string Replace characters at an arbitrary index with other characters Expand or shrink the string (truncates or adds characters at end of string) Swaps the value of two strings |
| **Input and Output**                                         |                                                              |
| >>, getline() << [c_str()](https://www.learncpp.com/cpp-tutorial/17-4-stdstring-character-access-and-conversion-to-c-style-arrays/) [copy()](https://www.learncpp.com/cpp-tutorial/17-4-stdstring-character-access-and-conversion-to-c-style-arrays/) [data()](https://www.learncpp.com/cpp-tutorial/17-4-stdstring-character-access-and-conversion-to-c-style-arrays/) | Reads values from the input stream into the string Writes string value to the output stream Returns the contents of the string as a NULL-terminated C-style string Copies contents (not NULL-terminated) to a character array Same as c_str(). The non-const overload allows writing to the returned string. |
| **String comparison**                                        |                                                              |
| ==, != <, <=, > >= compare()                                 | Compares whether two strings are equal/unequal (returns bool) Compares whether two strings are less than / greater than each other (returns bool) Compares whether two strings are equal/unequal (returns -1, 0, or 1) |
| **Substrings and concatenation**                             |                                                              |
| + substr()                                                   | Concatenates two strings Returns a substring                 |
| **Searching**                                                |                                                              |
| find() find_first_of() find_first_not_of() find_last_of() find_last_not_of() rfind() | Find index of first character/substring Find index of first character from a set of characters Find index of first character not from a set of characters Find index of last character from a set of characters Find index of last character not from a set of characters Find index of last character/substring |
| **Iterator and allocator support**                           |                                                              |
| begin(), end() get_allocator() rbegin(), rend()              | Forward-direction iterator support for beginning/end of string Returns the allocator Reverse-direction iterator support for beginning/end of string |

While the standard library string classes provide a lot of functionality, there are a few notable omissions:

- Regular expression support
- Constructors for creating strings from numbers
- Capitalization / upper case / lower case functions
- Case-insensitive comparisons
- Tokenization / splitting string into array
- Easy functions for getting the left or right hand portion of string
- Whitespace trimming
- Formatting a string sprintf style
- Conversion from utf-8 to utf-16 or vice-versa

For most of these, you will have to either write your own functions, or convert your string to a C-style string (using c_str()) and use the C functions that offer this functionality.

In the next lessons, we will look at the various functions of the string class in more depth. Although we will use string for our examples, everything is equally applicable to wstring.

------

### 22.2 — std::string construction and destruction

In this lesson, we’ll take a look at how to construct objects of std::string, as well as how to create strings from numbers and vice-versa.

#### **String construction**

The string classes have a number of constructors that can be used to create strings. We’ll take a look at each of them here.

Note: string::size_type resolves to size_t, which is the same unsigned integral type that is returned by the sizeof operator. The actual size of size_t depending on the environment. For the purposes of this tutorial, envision it as an unsigned int.

**string::string()**

- This is the default constructor. It creates an empty string.

Sample code:

```cpp
std::string sSource;
std::cout << sSource;
```

Output:

```

```

**string::string(const string& strString)**

- This is the copy constructor. This constructor creates a new string as a copy of strString.

Sample code:

```cpp
std::string sSource{ "my string" };
std::string sOutput{ sSource };
std::cout << sOutput;
```

Output:

```
my string
```

**string::string(const string& strString, size_type unIndex)** 

**string::string(const string& strString, size_type unIndex, size_type unLength)**

- This constructor creates a new string that contains at most unLength characters from strString, starting with index unIndex. If a NULL is encountered, the string copy will end, even if unLength has not been reached.
- If no unLength is supplied, all characters starting from unIndex will be used.
- If unIndex is larger than the size of the string, the out_of_range exception will be thrown.

Sample code:

```cpp
std::string sSource{ "my string" };
std::string sOutput{ sSource, 3 };
std::cout << sOutput<< '\n';
std::string sOutput2(sSource, 3, 4);
std::cout << sOutput2 << '\n';
```

Output:

```
string
stri
```

**string::string(const char\* szCString)**

- This constructor creates a new string from the C-style string szCString, up to but not including the NULL terminator.
- If the resulting size exceeds the maximum string length, the length_error exception will be thrown.
- Warning: szCString must not be NULL.

Sample code:

```cpp
const char* szSource{ "my string" };
std::string sOutput{ szSource };
std::cout << sOutput << '\n';
```

Output:

```
my string
```

**string::string(const char\* szCString, size_type unLength)**

- This constructor creates a new string from the first unLength chars from the C-style string szCString.
- If the resulting size exceeds the maximum string length, the length_error exception will be thrown.
- Warning: For this function only, NULLs are not treated as end-of-string characters in szCString! This means it is possible to read off the end of your string if unLength is too big. Be careful not to overflow your string buffer!

Sample code:

```cpp
const char* szSource{ "my string" };
std::string sOutput(szSource, 4);
std::cout << sOutput << '\n';
```

Output:

```
my s
```

**string::string(size_type nNum, char chChar)**

- This constructor creates a new string initialized by nNum occurances of the character chChar.
- If the resulting size exceeds the maximum string length, the length_error exception will be thrown.

Sample code:

```cpp
std::string sOutput(4, 'Q');
std::cout << sOutput << '\n';
```

Output:

```
QQQQ
```

**template string::string(InputIterator itBeg, InputIterator itEnd)**

- This constructor creates a new string initialized by the characters of range [itBeg, itEnd).
- If the resulting size exceeds the maximum string length, the length_error exception will be thrown.

No sample code for this one. It’s obscure enough you’ll probably never use it.

**string::~string()**

String destruction

- This is the destructor. It destroys the string and frees the memory.

No sample code here either since the destructor isn’t called explicitly.

------

#### **Constructing strings from numbers**

One notable omission in the std::string class is the lack of ability to create strings from numbers. For example:

```cpp
std::string sFour{ 4 };
```

Produces the following error:

```
c:vcprojectstest2test2test.cpp(10) : error C2664: 'std::basic_string<_Elem,_Traits,_Ax>::basic_string(std::basic_string<_Elem,_Traits,_Ax>::_Has_debug_it)' : cannot convert parameter 1 from 'int' to 'std::basic_string<_Elem,_Traits,_Ax>::_Has_debug_it'
```

Remember what I said about the string classes producing horrible looking errors? The relevant bit of information here is:

```
cannot convert parameter 1 from 'int' to 'std::basic_string
```

In other words, it tried to convert your int into a string but failed.

The easiest way to convert numbers into strings is to involve the std::ostringstream class. std::ostringstream is already set up to accept input from a variety of sources, including characters, numbers, strings, etc… It is also capable of outputting strings (either via the extraction operator>>, or via the str() function). For more information on std::ostringstream, see [23.4 -- Stream classes for strings](https://www.learncpp.com/cpp-tutorial/stream-classes-for-strings/).

Here’s a simple solution for creating std::string from various types of inputs:

```cpp
#include <iostream>
#include <sstream>
#include <string>

template <typename T>
inline std::string ToString(T tX)
{
    std::ostringstream oStream;
    oStream << tX;
    return oStream.str();
}
```

Here’s some sample code to test it:

```cpp
int main()
{
    std::string sFour{ ToString(4) };
    std::string sSixPointSeven{ ToString(6.7) };
    std::string sA{ ToString('A') };
    std::cout << sFour << '\n';
    std::cout << sSixPointSeven << '\n';
    std::cout << sA << '\n';
}
```

And the output:

```
4
6.7
A
```

Note that this solution omits any error checking. It is possible that inserting tX into oStream could fail. An appropriate response would be to throw an exception if the conversion fails.

**Related content**

The standard library also contains a function named `std::to_string()` that can be used to convert numbers into a std::string. While this is a simpler solution for basic cases, the output of std::to_string may differ from the output of std::cout or our ToString() function above. Some of these differences are currently documented [here](https://en.cppreference.com/w/cpp/string/basic_string/to_string).

------

#### **Converting strings to numbers**

Similar to the solution above:

```cpp
#include <iostream>
#include <sstream>
#include <string>

template <typename T>
inline bool FromString(const std::string& sString, T& tX)
{
    std::istringstream iStream(sString);
    return !(iStream >> tX).fail(); // extract value into tX, return success or not
}
```

Here’s some sample code to test it:

```cpp
int main()
{
    double dX;
    if (FromString("3.4", dX))
        std::cout << dX << '\n';
    if (FromString("ABC", dX))
        std::cout << dX << '\n';
}
```

And the output:

```
3.4
```

Note that the second conversion failed and returned false.

------

### 22.3 — std::string length and capacity

Once you’ve created strings, it’s often useful to know how long they are. This is where length and capacity operations come into play. We’ll also discuss various ways to convert std::string back into C-style strings, so you can use them with functions that expect strings of type char*.

#### **Length of a string**

The length of the string is quite simple -- it’s the number of characters in the string. There are two identical functions for determining string length:

**size_type string::length() const** 

**size_type string::size() const**

- Both of these functions return the current number of characters in the string, excluding the null terminator.

Sample code:

```cpp
std::string s { "012345678" };
std::cout << s.length() << '\n';
```

Output:

```
9
```

Although it’s possible to use length() to determine whether a string has any characters or not, it’s more efficient to use the empty() function:

**bool string::empty() const**

- Returns true if the string has no characters, false otherwise.

Sample code:

```cpp
std::string string1 { "Not Empty" };
std::cout << (string1.empty() ? "true" : "false") << '\n';
std::string string2; // empty
std::cout << (string2.empty() ? "true" : "false")  << '\n';
```

Output:

```
false
true
```

There is one more size-related function that you will probably never use, but we’ll include it here for completeness:

**size_type string::max_size() const**

- Returns the maximum number of characters that a string is allowed to have.
- This value will vary depending on operating system and system architecture.

Sample code:

```cpp
std::string s { "MyString" };
std::cout << s.max_size() << '\n';
```

Output:

```
4294967294
```

------

#### **Capacity of a string**

The capacity of a string reflects how much memory the string allocated to hold its contents. This value is measured in string characters, excluding the NULL terminator. For example, a string with capacity 8 could hold 8 characters.

**size_type string::capacity() const**

- Returns the number of characters a string can hold without reallocation.

Sample code:

```cpp
std::string s { "01234567" };
std::cout << "Length: " << s.length() << '\n';
std::cout << "Capacity: " << s.capacity() << '\n';
```

Output:

```
Length: 8
Capacity: 15
```

Note that the capacity is higher than the length of the string! Although our string was length 8, the string actually allocated enough memory for 15 characters! Why was this done?

The important thing to recognize here is that if a user wants to put more characters into a string than the string has capacity for, the string has to be reallocated to a larger capacity. For example, if a string had both length and capacity of 8, then adding any characters to the string would force a reallocation. By making the capacity larger than the actual string, this gives the user some buffer room to expand the string before reallocation needs to be done.

As it turns out, reallocation is bad for several reasons:

First, reallocating a string is comparatively expensive. First, new memory has to be allocated. Then each character in the string has to be copied to the new memory. This can take a long time if the string is big. Finally, the old memory has to be deallocated. If you are doing many reallocations, this process can slow your program down significantly.

Second, whenever a string is reallocated, the contents of the string change to a new memory address. This means all references, pointers, and iterators to the string become invalid!

Note that it’s not always the case that strings will be allocated with capacity greater than length. Consider the following program:

```cpp
std::string s { "0123456789abcde" };
std::cout << "Length: " << s.length() << '\n';
std::cout << "Capacity: " << s.capacity() << '\n';
```

This program outputs:

```
Length: 15
Capacity: 15
```

(Results may vary depending on compiler).

Let’s add one character to the string and watch the capacity change:

```cpp
std::string s("0123456789abcde");
std::cout << "Length: " << s.length() << '\n';
std::cout << "Capacity: " << s.capacity() << '\n';

// Now add a new character
s += "f";
std::cout << "Length: " << s.length() << '\n';
std::cout << "Capacity: " << s.capacity() << '\n';
```

This produces the result:

```
Length: 15
Capacity: 15
Length: 16
Capacity: 31
```

**void string::reserve()**
**void string::reserve(size_type unSize)**

- The second flavor of this function sets the capacity of the string to at least unSize (it can be greater). Note that this may require a reallocation to occur.
- If the first flavor of the function is called, or the second flavor is called with unSize less than the current capacity, the function will try to shrink the capacity to match the length. This request to shrink the capacity may be ignored, depending on implementation.

Sample code:

```cpp
std::string s { "01234567" };
std::cout << "Length: " << s.length() << '\n';
std::cout << "Capacity: " << s.capacity() << '\n';

s.reserve(200);
std::cout << "Length: " << s.length() << '\n';
std::cout << "Capacity: " << s.capacity() << '\n';

s.reserve();
std::cout << "Length: " << s.length() << '\n';
std::cout << "Capacity: " << s.capacity() << '\n';
```

Output:

```
Length: 8
Capacity: 15
Length: 8
Capacity: 207
Length: 8
Capacity: 207
```

This example shows two interesting things. First, although we requested a capacity of 200, we actually got a capacity of 207. The capacity is always guaranteed to be at least as large as your request, but may be larger. We then requested the capacity change to fit the string. This request was ignored, as the capacity did not change.

If you know in advance that you’re going to be constructing a large string by doing lots of string operations that will add to the size of the string, you can avoid having the string reallocated multiple times by reserving enough capacity from the outset:

```cpp
#include <iostream>
#include <string>
#include <cstdlib> // for rand() and srand()
#include <ctime> // for time()

int main()
{
    std::srand(std::time(nullptr)); // seed random number generator

    std::string s{}; // length 0
    s.reserve(64); // reserve 64 characters

    // Fill string up with random lower case characters
    for (int count{ 0 }; count < 64; ++count)
        s += 'a' + std::rand() % 26;

    std::cout << s;
}
```

The result of this program will change each time, but here’s the output from one execution:

```
wzpzujwuaokbakgijqdawvzjqlgcipiiuuxhyfkdppxpyycvytvyxwqsbtielxpy
```

Rather than having to reallocate s multiple times, we set the capacity once and then fill the string up. This can make a huge difference in performance when constructing large strings via concatenation.

------

### 22.4 — std::string character access and conversion to C-style arrays

#### **Character access**

There are two almost identical ways to access characters in a string. The easier to use and faster version is the overloaded operator[]:

**char& string::operator[] (size_type nIndex)**
**const char& string::operator[] (size_type nIndex) const**

- Both of these functions return the character with index nIndex
- Passing an invalid index results in undefined behavior
- Because char& is the return type, you can use this to edit characters in the array

Sample code:

```cpp
std::string sSource{ "abcdefg" };
std::cout << sSource[5] << '\n';
sSource[5] = 'X';
std::cout << sSource << '\n';
```

Output:

```
f
abcdeXg
```

There is also a non-operator version. This version is slower since it uses exceptions to check if the nIndex is valid. If you are not sure whether nIndex is valid, you should use this version to access the array:

**char& string::at (size_type nIndex)**
**const char& string::at (size_type nIndex) const**

- Both of these functions return the character with index nIndex
- Passing an invalid index results in an out_of_range exception
- Because char& is the return type, you can use this to edit characters in the array

Sample code:

```cpp
std::string sSource{ "abcdefg" };
std::cout << sSource.at(5) << '\n';
sSource.at(5) = 'X';
std::cout << sSource << '\n';
```

Output:

```
f
abcdeXg
```

------

#### **Conversion to C-style arrays**

Many functions (including all C functions) expect strings to be formatted as C-style strings rather than std::string. For this reason, std::string provides 3 different ways to convert std::string to C-style strings.

**const char\* string::c_str () const**

- Returns the contents of the string as a const C-style string
- A null terminator is appended
- The C-style string is owned by the std::string and should not be deleted

Sample code:

```cpp
#include <cstring>

std::string sSource{ "abcdefg" };
std::cout << std::strlen(sSource.c_str());
```

Output:

```
7
```

**const char\* string::data () const**

- Returns the contents of the string as a const C-style string
- A null terminator is appended. This function performs the same action as `c_str()`
- The C-style string is owned by the std::string and should not be deleted

Sample code:

```cpp
#include <cstring>

std::string sSource{ "abcdefg" };
const char* szString{ "abcdefg" };
// memcmp compares the first n characters of two C-style strings and returns 0 if they are equal
if (std::memcmp(sSource.data(), szString, sSource.length()) == 0)
    std::cout << "The strings are equal";
else
    std::cout << "The strings are not equal";
```

Output:

```
The strings are equal
```

**size_type string::copy(char\* szBuf, size_type nLength, size_type nIndex = 0) const**

- Both flavors copy at most nLength characters of the string to szBuf, beginning with character nIndex
- The number of characters copied is returned
- No null is appended. It is up to the caller to ensure szBuf is initialized to NULL or terminate the string using the returned length
- The caller is responsible for not overflowing szBuf

Sample code:

```cpp
std::string sSource{ "sphinx of black quartz, judge my vow" };

char szBuf[20];
int nLength{ static_cast<int>(sSource.copy(szBuf, 5, 10)) };
szBuf[nLength] = '\0';  // Make sure we terminate the string in the buffer

std::cout << szBuf << '\n';
```

Output:

```
black
```

This function should be avoided where possible as it is relatively dangerous (as it is up to the caller to provide null-termination and avoid buffer overflows).

------

### 22.5 — std::string assignment and swapping

#### **String assignment**

The easiest way to assign a value to a string is to use the overloaded operator= function. There is also an assign() member function that duplicates some of this functionality.

**string& string::operator= (const string& str)**
**string& string::assign (const string& str)**
**string& string::operator= (const char\* str)**
**string& string::assign (const char\* str)**
**string& string::operator= (char c)**

- These functions assign values of various types to the string.
- These functions return *this so they can be “chained”.
- Note that there is no assign() function that takes a single char.

Sample code:

```cpp
std::string sString;

// Assign a string value
sString = std::string("One");
std::cout << sString << '\n';

const std::string sTwo("Two");
sString.assign(sTwo);
std::cout << sString << '\n';

// Assign a C-style string
sString = "Three";
std::cout << sString << '\n';

sString.assign("Four");
std::cout << sString << '\n';

// Assign a char
sString = '5';
std::cout << sString << '\n';

// Chain assignment
std::string sOther;
sString = sOther = "Six";
std::cout << sString << ' ' << sOther << '\n';
```

Output:

```
One
Two
Three
Four
5
Six Six
```

The assign() member function also comes in a few other flavors:

**string& string::assign (const string& str, size_type index, size_type len)**

- Assigns a substring of str, starting from index, and of length len
- Throws an out_of_range exception if the index is out of bounds
- Returns *this so it can be “chained”.

Sample code:

```cpp
const std::string sSource("abcdefg");
std::string sDest;

sDest.assign(sSource, 2, 4); // assign a substring of source from index 2 of length 4
std::cout << sDest << '\n';
```

Output:

```
cdef
```

**string& string::assign (const char\* chars, size_type len)**

- Assigns len characters from the C-style array chars
- Throws an length_error exception if the result exceeds the maximum number of characters
- Returns *this so it can be “chained”.

Sample code:

```cpp
std::string sDest;

sDest.assign("abcdefg", 4);
std::cout << sDest << '\n';
```

Output:

```
abcd
```

This function is potentially dangerous and its use is not recommended.

**string& string::assign (size_type len, char c)**

- Assigns len occurrences of the character c
- Throws a length_error exception if the result exceeds the maximum number of characters
- Returns *this so it can be “chained”.

Sample code:

```cpp
std::string sDest;

sDest.assign(4, 'g');
std::cout << sDest << '\n';
```

Output:

```
gggg
```

------

#### Swapping

If you have two strings and want to swap their values, there are two functions both named swap() that you can use.

**void string::swap (string& str)**
**void swap (string& str1, string& str2)**

- Both functions swap the value of the two strings. The member function swaps *this and str, the global function swaps str1 and str2.
- These functions are efficient and should be used instead of assignments to perform a string swap.

Sample code:

```cpp
std::string sStr1("red");
std::string sStr2("blue");

std::cout << sStr1 << ' ' << sStr2 << '\n';
swap(sStr1, sStr2);
std::cout << sStr1 << ' ' << sStr2 << '\n';
sStr1.swap(sStr2);
std::cout << sStr1 << ' ' << sStr2 << '\n';
```

Output:

```
red blue
blue red
red blue
```

------

### 22.6 — std::string appending

#### Appending

Appending strings to the end of an existing string is easy using either operator+=, append(), or push_back().

**string& string::operator+= (const string& str)**
**string& string::append (const string& str)**

- Both functions append the characters of str to the string.
- Both functions return *this so they can be “chained”.
- Both functions throw a length_error exception if the result exceeds the maximum number of characters.

Sample code:

```cpp
std::string sString{"one"};

sString += std::string{" two"};

std::string sThree{" three"};
sString.append(sThree);

std::cout << sString << '\n';
```

Output:

```
one two three
```

There’s also a flavor of append() that can append a substring:

**string& string::append (const string& str, size_type index, size_type num)**

- This function appends num characters from str, starting at index, to the string.
- Returns *this so it can be “chained”.
- Throws an out_of_range if index is out of bounds
- Throws a length_error exception if the result exceeds the maximum number of characters.

Sample code:

```cpp
std::string sString{"one "};

const std::string sTemp{"twothreefour"};
sString.append(sTemp, 3, 5); // append substring of sTemp starting at index 3 of length 5
std::cout << sString << '\n';
```

Output:

```
one three
```

Operator+= and append() also have versions that work on C-style strings:

**string& string::operator+= (const char\* str)**
**string& string::append (const char\* str)**

- Both functions append the characters of str to the string.
- Both functions return *this so they can be “chained”.
- Both functions throw a length_error exception if the result exceeds the maximum number of characters.
- str should not be NULL.

Sample code:

```cpp
std::string sString{"one"};

sString += " two";
sString.append(" three");
std::cout << sString << '\n';
```

Output:

```
one two three
```

There is an additional flavor of append() that works on C-style strings:

**string& string::append (const char\* str, size_type len)**

- Appends the first len characters of str to the string.
- Returns *this so they can be “chained”.
- Throw a length_error exception if the result exceeds the maximum number of characters.
- Ignores special characters (including ”)

Sample code:

```cpp
std::string sString{"one "};

sString.append("threefour", 5);
std::cout << sString << '\n';
```

Output:

```
one three
```

This function is dangerous and its use is not recommended.

There is also a set of functions that append characters. Note that the name of the non-operator function to append a character is push_back(), not append()!

**string& string::operator+= (char c)**
**void string::push_back (char c)**

- Both functions append the character c to the string.
- Operator += returns *this so it can be “chained”.
- Both functions throw a length_error exception if the result exceeds the maximum number of characters.

Sample code:

```cpp
std::string sString{"one"};

sString += ' ';
sString.push_back('2');
std::cout << sString << '\n';
```

Output:

```
one 2
```

Now you might be wondering why the name of the function is push_back() and not append(). This follows a naming convention used for stacks, where push_back() is the function that adds a single item to the end of the stack. If you envision a string as a stack of characters, using push_back() to add a single character to the end makes sense. However, the lack of an append() function is inconsistent in my view!

It turns out there is an append() function for characters, that looks like this:

**string& string::append (size_type num, char c)**

- Adds num occurrences of the character c to the string
- Returns *this so it can be “chained”.
- Throws a length_error exception if the result exceeds the maximum number of characters.

Sample code:

```cpp
std::string sString{"aaa"};

sString.append(4, 'b');
std::cout << sString << '\n';
```

Output:

```
aaabbbb
```

There’s one final flavor of append() that works with iterators:

**string& string::append (InputIterator start, InputIterator end)**

- Appends all characters from the range [start, end) (including start up to but not including end)
- Returns *this so it can be “chained”.
- Throws a length_error exception if the result exceeds the maximum number of characters.

------

### 22.7 — std::string inserting

#### Inserting

Inserting characters into an existing string can be done via the insert() function.

**string& string::insert (size_type index, const string& str)**
**string& string::insert (size_type index, const char\* str)**

- Both functions insert the characters of str into the string at index
- Both function return *this so they can be “chained”.
- Both functions throw out_of_range if index is invalid
- Both functions throw a length_error exception if the result exceeds the maximum number of characters.
- In the C-style string version, str must not be NULL.

Sample code:

```cpp
string sString("aaaa");
cout << sString << endl;

sString.insert(2, string("bbbb"));
cout << sString << endl;

sString.insert(4, "cccc");
cout << sString << endl;
```

Output:

```
aaaa
aabbbbaa
aabbccccbbaa
```

Here’s a crazy version of insert() that allows you to insert a substring into a string at an arbitrary index:

**string& string::insert (size_type index, const string& str, size_type startindex, size_type num)**

- This function inserts num characters str, starting from startindex, into the string at index.
- Returns *this so it can be “chained”.
- Throws an out_of_range if index or startindex is out of bounds
- Throws a length_error exception if the result exceeds the maximum number of characters.

Sample code:

```cpp
string sString("aaaa");

const string sInsert("01234567");
sString.insert(2, sInsert, 3, 4); // insert substring of sInsert from index [3,7) into sString at index 2
cout << sString << endl;
```

Output:

```
aa3456aa
```

There is a flavor of insert() that inserts the first portion of a C-style string:

**string& string::insert(size_type index, const char\* str, size_type len)**

- Inserts len characters of str into the string at index
- Returns *this so it can be “chained”.
- Throws an out_of_range exception if the index is invalid
- Throws a length_error exception if the result exceeds the maximum number of characters.
- Ignores special characters (such as ”)

Sample code:

```cpp
string sString("aaaa");

sString.insert(2, "bcdef", 3);
cout << sString << endl;
```

Output:

```
aabcdaa
```

There’s also a flavor of insert() that inserts the same character multiple times:

**string& string::insert(size_type index, size_type num, char c)**

- Inserts num instances of char c into the string at index
- Returns *this so it can be “chained”.
- Throws an out_of_range exception if the index is invalid
- Throws a length_error exception if the result exceeds the maximum number of characters.

Sample code:

```cpp
string sString("aaaa");

sString.insert(2, 4, 'c');
cout << sString << endl;
```

Output:

```
aaccccaa
```

And finally, the insert() function also has three different versions that use iterators:

**void insert(iterator it, size_type num, char c)**
**iterator string::insert(iterator it, char c)**
**void string::insert(iterator it, InputIterator begin, InputIterator end)**

- The first function inserts num instances of the character c before the iterator it.
- The second inserts a single character c before the iterator it, and returns an iterator to the position of the character inserted.
- The third inserts all characters between [begin,end) before the iterator it.
- All functions throw a length_error exception if the result exceeds the maximum number of characters.

------

