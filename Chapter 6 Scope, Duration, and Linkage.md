## **Scope, Duration, and Linkage**

### 6.1 — Compound statements (blocks)

A **compound statement** (also called a **block**, or **block statement**) is a group of *zero or more statements* that is treated by the compiler as if it were a single statement.

Blocks begin with a `{` symbol, end with a `}` symbol, with the statements to be executed being placed in between. Blocks can be used anywhere a single statement is allowed. No semicolon is needed at the end of a block.

You have already seen an example of blocks when writing functions, as the function body is a block:

```cpp
int add(int x, int y)
{ // start block
    return x + y;
} // end block (no semicolon)

int main()
{ // start block

    // multiple statements
    int value {}; // this is initialization, not a block
    add(3, 4);

    return 0;

} // end block (no semicolon)
```

------

#### Blocks inside other blocks

Although functions can’t be nested inside other functions, blocks *can be* nested inside other blocks:

```cpp
int add(int x, int y)
{ // block
    return x + y;
} // end block

int main()
{ // outer block

    // multiple statements
    int value {};

    { // inner/nested block
        add(3, 4);
    } // end inner/nested block

    return 0;

} // end outer block
```

When blocks are nested, the enclosing block is typically called the **outer block** and the enclosed block is called the **inner block** or **nested block**.

------

#### Using blocks to execute multiple statements conditionally

One of the most common use cases for blocks is in conjunction with `if statements`. By default, an `if statement` executes a single statement if the condition evaluates to `true`. However, we can replace this single statement with a block of statements if we want multiple statements to execute when the condition evaluates to `true`.

For example:

```cpp
#include <iostream>

int main()
{ // start of outer block
    std::cout << "Enter an integer: ";
    int value {};
    std::cin >> value;

    if (value >= 0)
    { // start of nested block
        std::cout << value << " is a positive integer (or zero)\n";
        std::cout << "Double this number is " << value * 2 << '\n';
    } // end of nested block
    else
    { // start of another nested block
        std::cout << value << " is a negative integer\n";
        std::cout << "The positive of this number is " << -value << '\n';
    } // end of another nested block

    return 0;
} // end of outer block
```

If the user enters the number 3, this program prints:

```
Enter an integer: 3
3 is a positive integer (or zero)
Double this number is 6
```

If the user enters the number -4, this program prints:

```
Enter an integer: -4
-4 is a negative integer
The positive of this number is 4
```

------

#### Block nesting levels

It is even possible to put blocks inside of blocks inside of blocks:

```cpp
#include <iostream>

int main()
{ // block 1, nesting level 1
    std::cout << "Enter an integer: ";
    int value {};
    std::cin >> value;

    if (value >  0)
    { // block 2, nesting level 2
        if ((value % 2) == 0)
        { // block 3, nesting level 3
            std::cout << value << " is positive and even\n";
        }
        else
        { // block 4, also nesting level 3
            std::cout << value << " is positive and odd\n";
        }
    }

    return 0;
}
```

The **nesting level** (also called the **nesting depth**) of a function is the maximum number of nested blocks you can be inside at any point in the function (including the outer block). In the above function, there are 4 blocks, but the nesting level is 3 since in this program you can never be inside more than 3 blocks at any point.

The C++ standard says that C++ compilers should support 256 levels of nesting -- however not all do (e.g. as of the time of writing, Visual Studio supports less).

It’s a good idea to keep your nesting level to 3 or less. Just as overly-long functions are good candidates for refactoring (breaking into smaller functions), overly-nested blocks are hard to read and are good candidates for refactoring (with the most-nested blocks becoming separate functions).

**Best practice**

Keep the nesting level of your functions to 3 or less. If your function has a need for more nested levels, consider refactoring your function into sub-functions.

------

### 6.2 — User-defined namespaces and the scope resolution operator

In lesson [2.9 -- Naming collisions and an introduction to namespaces](https://www.learncpp.com/cpp-tutorial/naming-collisions-and-an-introduction-to-namespaces/), we introduced the concept of `naming collisions` and `namespaces`. As a reminder, a naming collision occurs when two identical identifiers are introduced into the same scope, and the compiler can’t disambiguate which one to use. When this happens, compiler or linker will produce an error because they do not have enough information to resolve the ambiguity. As programs become larger, the number of identifiers increases linearly, which in turn causes the probability of a naming collision occurring to increase exponentially.

Let’s revisit an example of a naming collision, and then show how we can resolve it using namespaces. In the following example, `foo.cpp` and `goo.cpp` are the source files that contain functions that do different things but have the same name and parameters.

foo.cpp:

```cpp
// This doSomething() adds the value of its parameters
int doSomething(int x, int y)
{
    return x + y;
}
```

goo.cpp:

```cpp
// This doSomething() subtracts the value of its parameters
int doSomething(int x, int y)
{
    return x - y;
}
```

main.cpp:

```cpp
#include <iostream>

int doSomething(int x, int y); // forward declaration for doSomething

int main()
{
    std::cout << doSomething(4, 3) << '\n'; // which doSomething will we get?
    return 0;
}
```

If this project contains only `foo.cpp` *or* `goo.cpp` (but not both), it will compile and run without incident. However, by compiling both into the same program, we have now introduced two different functions with the same name and parameters into the same scope (the global scope), which causes a naming collision. As a result, the linker will issue an error:

```
goo.cpp:3: multiple definition of `doSomething(int, int)'; foo.cpp:3: first defined here
```

Note that this error happens at the point of redefinition, so it doesn’t matter whether function `doSomething` is ever called.

One way to resolve this would be to rename one of the functions, so the names no longer collide. But this would also require changing the names of all the function calls, which can be a pain, and is subject to error. A better way to avoid collisions is to put your functions into your own namespaces. For this reason the standard library was moved into the `std` namespace.

------

#### Defining your own namespaces

C++ allows us to define our own namespaces via the `namespace` keyword. Namespaces that you create in your own programs are casually called **user-defined namespaces** (though it would be more accurate to call them **program-defined namespaces**).

Historically, namespace names have not been capitalized, and many style guides still recommend this convention. However, in modern C++, it is acceptable to capitalize namespace names.

**For advanced readers**

Some reasons to prefer namespace names starting with a capital letter:

- It is convention to name program-defined types starting with a capital letter. Using the same convention for program-defined namespaces is consistent (especially when using a qualified name such as `Foo::x`).
- It helps prevent naming collisions with other system-provided or library-provided lower-cased names.
- The C++20 standards document uses this style.

For now, we will consider either style to be acceptable.

Here is an example of the files in the prior example rewritten using namespaces:

foo.cpp:

```cpp
namespace foo // define a namespace named foo
{
    // This doSomething() belongs to namespace foo
    int doSomething(int x, int y)
    {
        return x + y;
    }
}
```

goo.cpp:

```cpp
namespace goo // define a namespace named goo
{
    // This doSomething() belongs to namespace goo
    int doSomething(int x, int y)
    {
        return x - y;
    }
}
```

Now `doSomething()` inside of `foo.cpp` is inside the `foo` namespace, and the `doSomething()` inside of `goo.cpp` is inside the `goo` namespace. Let’s see what happens when we recompile our program.

main.cpp:

```cpp
int doSomething(int x, int y); // forward declaration for doSomething

int main()
{
    std::cout << doSomething(4, 3) << '\n'; // which doSomething will we get?
    return 0;
}
```

The answer is that we now get another error!

```
ConsoleApplication1.obj : error LNK2019: unresolved external symbol "int __cdecl doSomething(int,int)" (?doSomething@@YAHHH@Z) referenced in function _main
```

In this case, the compiler was satisfied (by our forward declaration), but the linker could not find a definition for `doSomething` in the global namespace. This is because both of our versions of `doSomething` are no longer in the global namespace!

There are two different ways to tell the compiler which version of `doSomething()` to use, via the `scope resolution operator`, or via `using statements` (which we’ll discuss in a later lesson in this chapter).

For the subsequent examples, we’ll collapse our examples down to a one-file solution for ease of reading.

------

#### Accessing a namespace with the scope resolution operator (::)

The best way to tell the compiler to look in a particular namespace for an identifier is to use the **scope resolution operator** (::). The scope resolution operator tells the compiler that the identifier specified by the right-hand operand should be looked for in the scope of the left-hand operand.

Here is an example of using the scope resolution operator to tell the compiler that we explicitly want to use the version of `doSomething()` that lives in the `foo` namespace:

```cpp
#include <iostream>

namespace foo // define a namespace named foo
{
    // This doSomething() belongs to namespace foo
    int doSomething(int x, int y)
    {
        return x + y;
    }
}

namespace goo // define a namespace named goo
{
    // This doSomething() belongs to namespace goo
    int doSomething(int x, int y)
    {
        return x - y;
    }
}

int main()
{
    std::cout << foo::doSomething(4, 3) << '\n'; // use the doSomething() that exists in namespace foo
    return 0;
}
```

This produces the expected result:

```
7
```

If we wanted to use the version of `doSomething()` that lives in `goo` instead:

```cpp
#include <iostream>

namespace foo // define a namespace named foo
{
    // This doSomething() belongs to namespace foo
    int doSomething(int x, int y)
    {
        return x + y;
    }
}

namespace goo // define a namespace named goo
{
    // This doSomething() belongs to namespace goo
    int doSomething(int x, int y)
    {
        return x - y;
    }
}

int main()
{
    std::cout << goo::doSomething(4, 3) << '\n'; // use the doSomething() that exists in namespace goo
    return 0;
}
```

This produces the result:

```
1
```

The scope resolution operator is great because it allows us to *explicitly* pick which namespace we want to look in, so there’s no potential ambiguity. We can even do the following:

```cpp
#include <iostream>

namespace foo // define a namespace named foo
{
    // This doSomething() belongs to namespace foo
    int doSomething(int x, int y)
    {
        return x + y;
    }
}

namespace goo // define a namespace named goo
{
    // This doSomething() belongs to namespace goo
    int doSomething(int x, int y)
    {
        return x - y;
    }
}

int main()
{
    std::cout << foo::doSomething(4, 3) << '\n'; // use the doSomething() that exists in namespace foo
    std::cout << goo::doSomething(4, 3) << '\n'; // use the doSomething() that exists in namespace goo
    return 0;
}
```

This produces the result:

```
7
1
```

------

#### Using the scope resolution operator with no name prefix

The scope resolution operator can also be used in front of an identifier without providing a namespace name (e.g. `::doSomething`). In such a case, the identifier (e.g. `doSomething`) is looked for in the global namespace.

```cpp
#include <iostream>

void print() // this print lives in the global namespace
{
	std::cout << " there\n";
}

namespace foo
{
	void print() // this print lives in the foo namespace
	{
		std::cout << "Hello";
	}
}

int main()
{
	foo::print(); // call print() in foo namespace
	::print(); // call print() in global namespace (same as just calling print() in this case)

	return 0;
}
```

In the above example, the `::print()` performs the same as if we’d called `print()` with no scope resolution, so use of the scope resolution operator is superfluous in this case. But the next example will show a case where the scope resolution operator with no namespace can be useful.

------

#### Identifier resolution from within a namespace

If an identifier inside a namespace is used and no scope resolution is provided, the compiler will first try to find a matching declaration in that same namespace. If no matching identifier is found, the compiler will then check each containing namespace in sequence to see if a match is found, with the global namespace being checked last.

```cpp
#include <iostream>

void print() // this print lives in the global namespace
{
	std::cout << " there\n";
}

namespace foo
{
	void print() // this print lives in the foo namespace
	{
		std::cout << "Hello";
	}

	void printHelloThere()
	{
		print(); // calls print() in foo namespace
		::print(); // calls print() in global namespace
	}
}

int main()
{
	foo::printHelloThere();

	return 0;
}
```

This prints:

```
Hello there
```

In the above example, `print()` is called with no scope resolution provided. Because this use of `print()` is inside the `foo` namespace, the compiler will first see if a declaration for `foo::print()` can be found. Since one exists, `foo::print()` is called.

If `foo::print()` had not been found, the compiler would have checked the containing namespace (in this case, the global namespace) to see if it could match a `print()` there.

Note that we also make use of the scope resolution operator with no namespace (`::print()`) to explicitly call the global version of `print()`.

------

#### Multiple namespace blocks are allowed

It’s legal to declare namespace blocks in multiple locations (either across multiple files, or multiple places within the same file). All declarations within the namespace are considered part of the namespace.

circle.h:

```cpp
#ifndef CIRCLE_H
#define CIRCLE_H

namespace basicMath
{
    constexpr double pi{ 3.14 };
}

#endif
```

growth.h:

```cpp
#ifndef GROWTH_H
#define GROWTH_H

namespace basicMath
{
    // the constant e is also part of namespace basicMath
    constexpr double e{ 2.7 };
}

#endif
```

main.cpp:

```cpp
#include "circle.h" // for basicMath::pi
#include "growth.h" // for basicMath::e

#include <iostream>

int main()
{
    std::cout << basicMath::pi << '\n';
    std::cout << basicMath::e << '\n';

    return 0;
}
```

This works exactly as you would expect:

```
3.14
2.7
```

The standard library makes extensive use of this feature, as each standard library header file contains its declarations inside a `namespace std` block contained within that header file. Otherwise the entire standard library would have to be defined in a single header file!

Note that this capability also means you could add your own functionality to the `std` namespace. Doing so causes undefined behavior most of the time, because the `std` namespace has a special rule, prohibiting extension from user code.

**Warning**

Do not add custom functionality to the std namespace.

When you separate your code into multiple files, you’ll have to use a namespace in the header and source file.

add.h

```cpp
#ifndef ADD_H
#define ADD_H

namespace basicMath
{
    // function add() is part of namespace basicMath
    int add(int x, int y);
}

#endif
```

add.cpp

```cpp
#include "add.h"

namespace basicMath
{
    // define the function add()
    int add(int x, int y)
    {
        return x + y;
    }
}
```

main.cpp

```cpp
#include "add.h" // for basicMath::add()

#include <iostream>

int main()
{
    std::cout << basicMath::add(4, 3) << '\n';

    return 0;
}
```

If the namespace is omitted in the source file, the linker won’t find a definition of `basicMath::add`, because the source file only defines `add` (global namespace). If the namespace is omitted in the header file, “main.cpp” won’t be able to use `basicMath::add`, because it only sees a declaration for `add` (global namespace).

------

#### Nested namespaces

Namespaces can be nested inside other namespaces. For example:

```cpp
#include <iostream>

namespace foo
{
    namespace goo // goo is a namespace inside the foo namespace
    {
        int add(int x, int y)
        {
            return x + y;
        }
    }
}

int main()
{
    std::cout << foo::goo::add(1, 2) << '\n';
    return 0;
}
```

Note that because namespace `goo` is inside of namespace `foo`, we access `add` as `foo::goo::add`.

Since C++17, nested namespaces can also be declared this way:

```cpp
#include <iostream>

namespace foo::goo // goo is a namespace inside the foo namespace (C++17 style)
{
  int add(int x, int y)
  {
    return x + y;
  }
}

int main()
{
    std::cout << foo::goo::add(1, 2) << '\n';
    return 0;
}
```

------

#### Namespace aliases

Because typing the qualified name of a variable or function inside a nested namespace can be painful, C++ allows you to create **namespace aliases**, which allow us to temporarily shorten a long sequence of namespaces into something shorter:

```cpp
#include <iostream>

namespace foo::goo
{
    int add(int x, int y)
    {
        return x + y;
    }
}

int main()
{
    namespace active = foo::goo; // active now refers to foo::goo

    std::cout << active::add(1, 2) << '\n'; // This is really foo::goo::add()

    return 0;
} // The active alias ends here
```

One nice advantage of namespace aliases: If you ever want to move the functionality within `foo::goo` to a different place, you can just update the `active` alias to reflect the new destination, rather than having to find/replace every instance of `foo::goo`.

```cpp
#include <iostream>

namespace foo::goo
{
}

namespace v2
{
    int add(int x, int y)
    {
        return x + y;
    }
}

int main()
{
    namespace active = v2; // active now refers to v2

    std::cout << active::add(1, 2) << '\n'; // We don't have to change this

    return 0;
}
```

It’s worth noting that namespaces in C++ were not originally designed as a way to implement an information hierarchy -- they were designed primarily as a mechanism for preventing naming collisions. As evidence of this, note that the entirety of the standard library lives under the singular namespace `std::` (with some nested namespaces used for newer library features). Some newer languages (such as C#) differ from C++ in this regard.

In general, you should avoid deeply nested namespaces.

------

#### When you should use namespaces

In applications, namespaces can be used to separate application-specific code from code that might be reusable later (e.g. math functions). For example, physical and math functions could go into one namespace (e.g. `math::`). Language and localization functions in another (e.g. `lang::`).

When you write a library or code that you want to distribute to others, always place your code inside a namespace. The code your library is used in may not follow best practices -- in such a case, if your library’s declarations aren’t in a namespace, there’s an elevated chance for naming conflicts to occur. As an additional advantage, placing library code inside a namespace also allows the user to see the contents of your library by using their editor’s auto-complete and suggestion feature.

------

### 6.3 — Local variables

In lesson [2.5 -- Introduction to local scope](https://www.learncpp.com/cpp-tutorial/introduction-to-local-scope/), we introduced `local variables`, which are variables that are defined inside a function (including function parameters).

It turns out that C++ actually doesn’t have a single attribute that defines a variable as being a local variable. Instead, local variables have several different properties that differentiate how local variables behave from other kinds of (non-local) variables. We’ll explore these properties in this and upcoming lessons.

In lesson [2.5 -- Introduction to local scope](https://www.learncpp.com/cpp-tutorial/introduction-to-local-scope/), we also introduced the concept of scope. An identifier’s `scope` determines where an identifier can be accessed within the source code. When an identifier can be accessed, we say it is `in scope`. When an identifier can not be accessed, we say it is `out of scope`. Scope is a compile-time property, and trying to use an identifier when it is out of scope will result in a compile error.

------

#### Local variables have block scope

Local variables have **block scope**, which means they are *in scope* from their point of definition to the end of the block they are defined within.

**Related content**

Please review lesson [6.1 -- Compound statements (blocks)](https://www.learncpp.com/cpp-tutorial/compound-statements-blocks/) if you need a refresher on blocks.

```cpp
int main()
{
    int i { 5 }; // i enters scope here
    double d { 4.0 }; // d enters scope here

    return 0;
} // d and i go out of scope here
```

Although function parameters are not defined inside the function body, for typical functions they can be considered to be part of the scope of the function body block.

```cpp
int max(int x, int y) // x and y enter scope here
{
    // assign the greater of x or y to max
    int max{ (x > y) ? x : y }; // max enters scope here

    return max;
} // max, y, and x leave scope here
```

------

#### All variable names within a scope must be unique

Variable names must be unique within a given scope, otherwise any reference to the name will be ambiguous. Consider the following program:

```cpp
void someFunction(int x)
{
    int x{}; // compilation failure due to name collision with function parameter
}

int main()
{
    return 0;
}
```

The above program doesn’t compile because the variable `x` defined inside the function body and the function parameter `x` have the same name and both are in the same block scope.

------

#### Local variables have automatic storage duration

A variable’s **storage duration** (usually just called **duration**) determines what rules govern when and how a variable will be created and destroyed. In most cases, a variable’s storage duration directly determines its `lifetime`.

**Related content**

We discuss what a lifetime is in lesson [2.5 -- Introduction to local scope](https://www.learncpp.com/cpp-tutorial/introduction-to-local-scope/).

For example, local variables have **automatic storage duration**, which means they are created at the point of definition and destroyed at the end of the block they are defined in. For example:

```cpp
int main()
{
    int i { 5 }; // i created and initialized here
    double d { 4.0 }; // d created and initialized here

    return 0;
} // d and i are destroyed here
```

For this reason, local variables are sometimes called **automatic variables**.

------

#### Local variables in nested blocks

Local variables can be defined inside nested blocks. This works identically to local variables in function body blocks:

```cpp
int main() // outer block
{
    int x { 5 }; // x enters scope and is created here

    { // nested block
        int y { 7 }; // y enters scope and is created here
    } // y goes out of scope and is destroyed here

    // y can not be used here because it is out of scope in this block

    return 0;
} // x goes out of scope and is destroyed here
```

In the above example, variable `y` is defined inside a nested block. Its scope is limited from its point of definition to the end of the nested block, and its lifetime is the same. Because the scope of variable `y` is limited to the inner block in which it is defined, it’s not accessible anywhere in the outer block.

Note that nested blocks are considered part of the scope of the outer block in which they are defined. Consequently, variables defined in the outer block *can* be seen inside a nested block:

```cpp
#include <iostream>

int main()
{ // outer block

    int x { 5 }; // x enters scope and is created here

    { // nested block
        int y { 7 }; // y enters scope and is created here

        // x and y are both in scope here
        std::cout << x << " + " << y << " = " << x + y << '\n';
    } // y goes out of scope and is destroyed here

    // y can not be used here because it is out of scope in this block

    return 0;
} // x goes out of scope and is destroyed here
```

------

#### Local variables have no linkage

Identifiers have another property named `linkage`. An identifier’s **linkage** determines whether other declarations of that name refer to the same object or not.

Local variables have `no linkage`, which means that each declaration refers to a unique object. For example:

```cpp
int main()
{
    int x { 2 }; // local variable, no linkage

    {
        int x { 3 }; // this identifier x refers to a different object than the previous x
    }

    return 0;
}
```

Scope and linkage may seem somewhat similar. However, scope defines where a single declaration can be seen and used. Linkage defines whether multiple declarations refer to the same object or not.

**Related content**

We discuss what happens when variables with the same name appear in nested blocks in lesson [6.5 -- Variable shadowing (name hiding)](https://www.learncpp.com/cpp-tutorial/variable-shadowing-name-hiding/).

Linkage isn’t very interesting in the context of local variables, but we’ll talk about it more in the next few lessons.

------

#### Variables should be defined in the most limited scope

If a variable is only used within a nested block, it should be defined inside that nested block:

```cpp
#include <iostream>

int main()
{
    // do not define y here

    {
        // y is only used inside this block, so define it here
        int y { 5 };
        std::cout << y << '\n';
    }

    // otherwise y could still be used here, where it's not needed

    return 0;
}
```

By limiting the scope of a variable, you reduce the complexity of the program because the number of active variables is reduced. Further, it makes it easier to see where variables are used (or aren’t used). A variable defined inside a block can only be used within that block (or nested blocks). This can make the program easier to understand.

If a variable is needed in an outer block, it needs to be declared in the outer block:

```cpp
#include <iostream>

int main()
{
    int y { 5 }; // we're declaring y here because we need it in this outer block later

    {
        int x{};
        std::cin >> x;

        // if we declared y here, immediately before its actual first use...
        if (x == 4)
            y = 4;
    } // ... it would be destroyed here

    std::cout << y; // and we need y to exist here

    return 0;
}
```

The above example shows one of the rare cases where you may need to declare a variable well before its first use.

New developers sometimes wonder whether it’s worth creating a nested block just to intentionally limit a variable’s scope (and force it to go out of scope / be destroyed early). Doing so makes that variable simpler, but the overall function becomes longer and more complex as a result. The tradeoff generally isn’t worth it. If creating a nested block seems useful to intentionally limit the scope of a chunk of code, that code might be better to put in a separate function instead.

**Best practice**

Define variables in the most limited existing scope. Avoid creating new blocks whose only purpose is to limit the scope of variables.

------

#### Quiz time

**Question #1**

Write a program that asks the user to enter two integers, one named `smaller`, the other named `larger`. If the user enters a smaller value for the second integer, use a block and a temporary variable to swap the smaller and larger values. Then print the values of the `smaller` and `larger` variables. Add comments to your code indicating where each variable dies. Note: When you print the values, `smaller` should hold the smaller input and `larger` the larger input, no matter which order they were entered in.

The program output should match the following:

```
Enter an integer: 4
Enter a larger integer: 2
Swapping the values
The smaller value is 2
The larger value is 4
```

```cpp
#include <iostream>

int main()
{
    std::cout << "Enter an integer: ";
    int smaller{};
    std::cin >> smaller;

    std::cout << "Enter a larger integer: ";
    int larger{};
    std::cin >> larger;

    // if user did it wrong
    if (smaller > larger)
    {
        // swap values of smaller and larger
        std::cout << "Swapping the values\n";

        int temp{ larger };
        larger = smaller;
        smaller = temp;
    } // temp dies here

    std::cout << "The smaller value is: " << smaller << '\n';
    std::cout << "The larger value is: " << larger << '\n';

    return 0;
} // smaller and larger die here
```

In the future, you can use `std::swap()` from the `<utility>` header to swap the values of two variables. For example

```cpp
int temp{ larger };
larger = smaller;
smaller = temp;

// is the same as
std::swap(larger, smaller);
```

------

**Question #2**

What’s the difference between a variable’s scope, duration, and lifetime? By default, what kind of scope and duration do local variables have (and what do those mean)?

A variable’s scope determines where the variable is accessible. Duration defines the rules that govern when a variable is created and destroyed. A variable’s lifetime is the actual time between its creation and destruction.

Local variables have block scope, which means they can be accessed inside the block in which they are defined.

Local variables have automatic duration, which means they are created at the point of definition, and destroyed at the end of the block in which they are defined.

------

### 6.4 — Introduction to global variables

In lesson [6.3 -- Local variables](https://www.learncpp.com/cpp-tutorial/local-variables/), we covered that local variables are variables defined inside a function (or function parameters). Local variables have block scope (are only visible within the block they are declared in), and have automatic duration (they are created at the point of definition and destroyed when the block is exited).

In C++, variables can also be declared *outside* of a function. Such variables are called **global variables**.

------

#### Declaring and naming global variables

By convention, global variables are declared at the top of a file, below the includes, but above any code. Here’s an example of a global variable being defined:

```cpp
#include <iostream>

// Variables declared outside of a function are global variables
int g_x {}; // global variable g_x

void doSomething()
{
    // global variables can be seen and used everywhere in the file
    g_x = 3;
    std::cout << g_x << '\n';
}

int main()
{
    doSomething();
    std::cout << g_x << '\n';

    // global variables can be seen and used everywhere in the file
    g_x = 5;
    std::cout << g_x << '\n';

    return 0;
}
// g_x goes out of scope here
```

The above example prints:

```
3
3
5
```

By convention, many developers prefix non-const global variable identifiers with “g” or “g_” to indicate that they are global.

**Best practice**

Consider using a “g” or “g_” prefix when naming non-const global variables, to help differentiate them from local variables and function parameters.

------

#### Global variables have file scope and static duration

Global variables have **file scope** (also informally called **global scope** or **global namespace scope**), which means they are visible from the point of declaration until the end of the *file* in which they are declared. Once declared, a global variable can be used anywhere in the file from that point onward! In the above example, global variable `g_x` is used in both functions `doSomething()` and `main()`.

Because they are defined outside of a function, global variables are considered to be part of the global namespace (hence the term “global namespace scope”).

Global variables are created when the program starts, and destroyed when it ends. This is called **static duration**. Variables with *static duration* are sometimes called **static variables**.

------

#### Global variable initialization

Unlike local variables, which are uninitialized by default, variables with static duration are zero-initialized by default.

Non-constant global variables can be optionally initialized:

```cpp
int g_x;       // no explicit initializer (zero-initialized by default)
int g_y {};    // zero-initialized
int g_z { 1 }; // initialized with value
```

------

#### Constant global variables

Just like local variables, global variables can be constant. As with all constants, constant global variables must be initialized.

```cpp
#include <iostream>

const int g_x; // error: constant variables must be initialized
constexpr int g_w; // error: constexpr variables must be initialized

const int g_y { 1 };  // const global variable g_y, initialized with a value
constexpr int g_z { 2 }; // constexpr global variable g_z, initialized with a value

void doSomething()
{
    // global variables can be seen and used everywhere in the file
    std::cout << g_y << '\n';
    std::cout << g_z << '\n';
}

int main()
{
    doSomething();

    // global variables can be seen and used everywhere in the file
    std::cout << g_y << '\n';
    std::cout << g_z << '\n';

    return 0;
}
// g_y and g_z goes out of scope here
```

**Related content**

We discuss global constants in more detail in lesson [6.9 -- Sharing global constants across multiple files (using inline variables)](https://www.learncpp.com/cpp-tutorial/sharing-global-constants-across-multiple-files-using-inline-variables/).

------

#### A word of caution about (non-constant) global variables

New programmers are often tempted to use lots of global variables, because they can be used without having to explicitly pass them to every function that needs them. However, use of non-constant global variables should generally be avoided altogether! We’ll discuss why in upcoming lesson [6.8 -- Why (non-const) global variables are evil](https://www.learncpp.com/cpp-tutorial/why-non-const-global-variables-are-evil/).

------

#### Quick Summary

```cpp
// Non-constant global variables
int g_x;                 // defines non-initialized global variable (zero initialized by default)
int g_x {};              // defines explicitly zero-initialized global variable
int g_x { 1 };           // defines explicitly initialized global variable

// Const global variables
const int g_y;           // error: const variables must be initialized
const int g_y { 2 };     // defines initialized global constant

// Constexpr global variables
constexpr int g_y;       // error: constexpr variables must be initialized
constexpr int g_y { 3 }; // defines initialized global const
```

------

### 6.5 — Variable shadowing (name hiding)

Each block defines its own scope region. So what happens when we have a variable inside a nested block that has the same name as a variable in an outer block? When this happens, the nested variable “hides” the outer variable in areas where they are both in scope. This is called **name hiding** or **shadowing**.

------

#### Shadowing of local variables

```cpp
#include <iostream>

int main()
{ // outer block
    int apples { 5 }; // here's the outer block apples

    { // nested block
        // apples refers to outer block apples here
        std::cout << apples << '\n'; // print value of outer block apples

        int apples{ 0 }; // define apples in the scope of the nested block

        // apples now refers to the nested block apples
        // the outer block apples is temporarily hidden

        apples = 10; // this assigns value 10 to nested block apples, not outer block apples

        std::cout << apples << '\n'; // print value of nested block apples
    } // nested block apples destroyed


    std::cout << apples << '\n'; // prints value of outer block apples

    return 0;
} // outer block apples destroyed
```

If you run this program, it prints:

```
5
10
5
```

In the above program, we first declare a variable named `apples` in the outer block. This variable is visible within the inner block, which we can see by printing its value (`5`). Then we declare a different variable (also named `apples`) in the nested block. From this point to the end of the block, the name `apples` refers to the nested block `apples`, not the outer block `apples`.

Thus, when we assign value `10` to `apples`, we’re assigning it to the nested block `apples`. After printing this value (`10`), the nested block ends and nested block `apples` is destroyed. The existence and value of outer block `apples` is not affected, and we prove this by printing the value of outer block `apples` (`5`).

Note that if the nested block `apples` had not been defined, the name `apples` in the nested block would still refer to the outer block `apples`, so the assignment of value `10` to `apples` would have applied to the outer block `apples`:

```cpp
#include <iostream>

int main()
{ // outer block
    int apples{5}; // here's the outer block apples

    { // nested block
        // apples refers to outer block apples here
        std::cout << apples << '\n'; // print value of outer block apples

        // no inner block apples defined in this example

        apples = 10; // this applies to outer block apples

        std::cout << apples << '\n'; // print value of outer block apples
    } // outer block apples retains its value even after we leave the nested block

    std::cout << apples << '\n'; // prints value of outer block apples

    return 0;
} // outer block apples destroyed
```

The above program prints:

```
5
10
10
```

When inside the nested block, there’s no way to directly access the shadowed variable from the outer block.

------

#### Shadowing of global variables

Similar to how variables in a nested block can shadow variables in an outer block, local variables with the same name as a global variable will shadow the global variable wherever the local variable is in scope:

```cpp
#include <iostream>
int value { 5 }; // global variable

void foo()
{
    std::cout << "global variable value: " << value << '\n'; // value is not shadowed here, so this refers to the global value
}

int main()
{
    int value { 7 }; // hides the global variable value until the end of this block

    ++value; // increments local value, not global value

    std::cout << "local variable value: " << value << '\n';

    foo();

    return 0;
} // local value is destroyed
```

This code prints:

```
local variable value: 8
global variable value: 5
```

However, because global variables are part of the global namespace, we can use the scope operator (::) with no prefix to tell the compiler we mean the global variable instead of the local variable.

```cpp
#include <iostream>
int value { 5 }; // global variable

int main()
{
    int value { 7 }; // hides the global variable value
    ++value; // increments local value, not global value

    --(::value); // decrements global value, not local value (parenthesis added for readability)

    std::cout << "local variable value: " << value << '\n';
    std::cout << "global variable value: " << ::value << '\n';

    return 0;
} // local value is destroyed
```

This code prints:

```
local variable value: 8
global variable value: 4
```

------

#### Avoid variable shadowing

Shadowing of local variables should generally be avoided, as it can lead to inadvertent errors where the wrong variable is used or modified. Some compilers will issue a warning when a variable is shadowed.

For the same reason that we recommend avoiding shadowing local variables, we recommend avoiding shadowing global variables as well. This is trivially avoidable if all of your global names use a “g_” prefix.

**Best practice**

Avoid variable shadowing.

**For GCC/G++ users**

GCC and Clang support the flag `-Wshadow` that will generate warnings if a variable is shadowed. There are several subvariants of this flag (`-Wshadow=global`, `-Wshadow=local`, and `-Wshadow=compatible-local`. Consult the [GCC documentation](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html) for an explanation of the differences.

------

### 6.6 — Internal linkage

In lesson [6.3 -- Local variables](https://www.learncpp.com/cpp-tutorial/local-variables/), we said, “An identifier’s linkage determines whether other declarations of that name refer to the same object or not”, and we discussed how local variables have `no linkage`.

Global variable and functions identifiers can have either `internal linkage` or `external linkage`. We’ll cover the internal linkage case in this lesson, and the external linkage case in lesson [6.7 -- External linkage and variable forward declarations](https://www.learncpp.com/cpp-tutorial/external-linkage-and-variable-forward-declarations/).

An identifier with **internal linkage** can be seen and used within a single translation unit, but it is not accessible from other translation units (that is, it is not exposed to the linker). This means that if two source files have identically named identifiers with internal linkage, those identifiers will be treated as independent (and not result in an ODR violation for having duplicate definitions).

**Related content**

We cover translation units in lesson [2.10 -- Introduction to the preprocessor](https://www.learncpp.com/cpp-tutorial/introduction-to-the-preprocessor/).

------

#### Global variables with internal linkage

Global variables with internal linkage are sometimes called **internal variables**.

To make a non-constant global variable internal, we use the `static` keyword.

```cpp
#include <iostream>

static int g_x{}; // non-constant globals have external linkage by default, but can be given internal linkage via the static keyword

const int g_y{ 1 }; // const globals have internal linkage by default
constexpr int g_z{ 2 }; // constexpr globals have internal linkage by default

int main()
{
    std::cout << g_x << ' ' << g_y << ' ' << g_z << '\n';
    return 0;
}
```

Const and constexpr global variables have internal linkage by default (and thus don’t need the `static` keyword -- if it is used, it will be ignored).

Here’s an example of multiple files using internal variables:

a.cpp:

```cpp
[[maybe_unused]] constexpr int g_x { 2 }; // this internal g_x is only accessible within a.cpp
```

main.cpp:

```cpp
#include <iostream>

static int g_x { 3 }; // this separate internal g_x is only accessible within main.cpp

int main()
{
    std::cout << g_x << '\n'; // uses main.cpp's g_x, prints 3

    return 0;
}
```

This program prints:

```
3
```

Because `g_x` is internal to each file, `main.cpp` has no idea that `a.cpp` also has a variable named `g_x` (and vice versa).

**For advanced readers**

The use of the `static` keyword above is an example of a **storage class specifier**, which sets both the name’s linkage and its storage duration. The most commonly used `storage class specifiers` are `static`, `extern`, and `mutable`. The term `storage class specifier` is mostly used in technical documentations.

------

#### The one-definition rule and internal linkage

In lesson [2.7 -- Forward declarations and definitions](https://www.learncpp.com/cpp-tutorial/forward-declarations/), we noted that the one-definition rule says that an object or function can’t have more than one definition, either within a file or a program.

However, it’s worth noting that internal objects (and functions) that are defined in different files are considered to be independent entities (even if their names and types are identical), so there is no violation of the one-definition rule. Each internal object only has one definition.

------

#### Functions with internal linkage

Because linkage is a property of an identifier (not of a variable), function identifiers have the same linkage property that variable identifiers do. Functions default to external linkage (which we’ll cover in the next lesson), but can be set to internal linkage via the `static` keyword:

add.cpp:

```cpp
// This function is declared as static, and can now be used only within this file
// Attempts to access it from another file via a function forward declaration will fail
[[maybe_unused]] static int add(int x, int y)
{
    return x + y;
}
```

main.cpp:

```cpp
#include <iostream>

int add(int x, int y); // forward declaration for function add

int main()
{
    std::cout << add(3, 4) << '\n';

    return 0;
}
```

This program won’t link, because function `add` is not accessible outside of `add.cpp`.

------

#### Quick Summary

```cpp
// Internal global variables definitions:
static int g_x;          // defines non-initialized internal global variable (zero initialized by default)
static int g_x{ 1 };     // defines initialized internal global variable

const int g_y { 2 };     // defines initialized internal global const variable
constexpr int g_y { 3 }; // defines initialized internal global constexpr variable

// Internal function definitions:
static int foo() {};     // defines internal function
```

We provide a comprehensive summary in lesson [6.11 -- Scope, duration, and linkage summary](https://www.learncpp.com/cpp-tutorial/scope-duration-and-linkage-summary/).

------

### 6.7 — External linkage and variable forward declarations

In the prior lesson ([6.6 -- Internal linkage](https://www.learncpp.com/cpp-tutorial/internal-linkage/)), we discussed how `internal linkage` limits the use of an identifier to a single file. In this lesson, we’ll explore the concept of `external linkage`.

An identifier with **external linkage** can be seen and used both from the file in which it is defined, and from other code files (via a forward declaration). In this sense, identifiers with external linkage are truly “global” in that they can be used anywhere in your program!

------

#### Functions have external linkage by default

In lesson [2.8 -- Programs with multiple code files](https://www.learncpp.com/cpp-tutorial/programs-with-multiple-code-files/), you learned that you can call a function defined in one file from another file. This is because functions have external linkage by default.

In order to call a function defined in another file, you must place a `forward declaration` for the function in any other files wishing to use the function. The forward declaration tells the compiler about the existence of the function, and the linker connects the function calls to the actual function definition.

Here’s an example:

a.cpp:

```cpp
#include <iostream>

void sayHi() // this function has external linkage, and can be seen by other files
{
    std::cout << "Hi!\n";
}
```

main.cpp:

```cpp
void sayHi(); // forward declaration for function sayHi, makes sayHi accessible in this file

int main()
{
    sayHi(); // call to function defined in another file, linker will connect this call to the function definition

    return 0;
}
```

The above program prints:

```
Hi!
```

In the above example, the forward declaration of function `sayHi()` in `main.cpp` allows `main.cpp` to access the `sayHi()` function defined in `a.cpp`. The forward declaration satisfies the compiler, and the linker is able to link the function call to the function definition.

If function `sayHi()` had internal linkage instead, the linker would not be able to connect the function call to the function definition, and a linker error would result.

------

#### Global variables with external linkage

Global variables with external linkage are sometimes called **external variables**. To make a global variable external (and thus accessible by other files), we can use the `extern` keyword to do so:

```cpp
int g_x { 2 }; // non-constant globals are external by default

extern const int g_y { 3 }; // const globals can be defined as extern, making them external
extern constexpr int g_z { 3 }; // constexpr globals can be defined as extern, making them external (but this is useless, see the note in the next section)

int main()
{
    return 0;
}
```

Non-const global variables are external by default (if used, the `extern` keyword will be ignored).

------

#### Variable forward declarations via the extern keyword

To actually use an external global variable that has been defined in another file, you also must place a `forward declaration` for the global variable in any other files wishing to use the variable. For variables, creating a forward declaration is also done via the `extern` keyword (with no initialization value).

Here is an example of using a variable forward declaration:

a.cpp:

```cpp
// global variable definitions
int g_x { 2 }; // non-constant globals have external linkage by default
extern const int g_y { 3 }; // this extern gives g_y external linkage
```

main.cpp:

```cpp
#include <iostream>

extern int g_x; // this extern is a forward declaration of a variable named g_x that is defined somewhere else
extern const int g_y; // this extern is a forward declaration of a const variable named g_y that is defined somewhere else

int main()
{
    std::cout << g_x << '\n'; // prints 2

    return 0;
}
```

In the above example, `a.cpp` and `main.cpp` both reference the same global variable named `g_x`. So even though `g_x` is defined and initialized in `a.cpp`, we are able to use its value in `main.cpp` via the forward declaration of `g_x`.

Note that the `extern` keyword has different meanings in different contexts. In some contexts, `extern` means “give this variable external linkage”. In other contexts, `extern` means “this is a forward declaration for an external variable that is defined somewhere else”. Yes, this is confusing, so we summarize all of these usages in lesson [6.11 -- Scope, duration, and linkage summary](https://www.learncpp.com/cpp-tutorial/scope-duration-and-linkage-summary/).

**Warning**

If you want to define an uninitialized non-const global variable, do not use the extern keyword, otherwise C++ will think you’re trying to make a forward declaration for the variable.

**Warning**

Although constexpr variables can be given external linkage via the `extern` keyword, they can not be forward declared, so there is no value in giving them external linkage.

This is because the compiler needs to know the value of the constexpr variable (at compile time). If that value is defined in some other file, the compiler has no visibility on what value was defined in that other file.

Note that function forward declarations don’t need the `extern` keyword -- the compiler is able to tell whether you’re defining a new function or making a forward declaration based on whether you supply a function body or not. Variables forward declarations *do* need the `extern` keyword to help differentiate variables definitions from variable forward declarations (they look otherwise identical):

```cpp
// non-constant
int g_x; // variable definition (can have initializer if desired)
extern int g_x; // forward declaration (no initializer)

// constant
extern const int g_y { 1 }; // variable definition (const requires initializers)
extern const int g_y; // forward declaration (no initializer)
```

------

#### File scope vs. global scope

The terms “file scope” and “global scope” tend to cause confusion, and this is partly due to the way they are informally used. Technically, in C++, *all* global variables have “file scope”, and the linkage property controls whether they can be used in other files or not.

Consider the following program:

global.cpp:

```cpp
int g_x { 2 }; // external linkage by default
// g_x goes out of scope here
```

main.cpp:

```cpp
#include <iostream>

extern int g_x; // forward declaration for g_x -- g_x can be used beyond this point in this file

int main()
{
    std::cout << g_x << '\n'; // should print 2

    return 0;
}
// the forward declaration for g_x goes out of scope here
```

Variable `g_x` has file scope within `global.cpp` -- it can be used from the point of definition to the end of the file, but it can not be directly seen outside of `global.cpp`.

Inside `main.cpp`, the forward declaration of `g_x` also has file scope -- it can be used from the point of declaration to the end of the file.

However, informally, the term “file scope” is more often applied to global variables with internal linkage, and “global scope” to global variables with external linkage (since they can be used across the whole program, with the appropriate forward declarations).

------

#### Quick summary

```cpp
// External global variable definitions:
int g_x;                       // defines non-initialized external global variable (zero initialized by default)
extern const int g_x{ 1 };     // defines initialized const external global variable
extern constexpr int g_x{ 2 }; // defines initialized constexpr external global variable

// Forward declarations
extern int g_y;                // forward declaration for non-constant global variable
extern const int g_y;          // forward declaration for const global variable
extern constexpr int g_y;      // not allowed: constexpr variables can't be forward declared
```

We provide a comprehensive summary in lesson [6.11 -- Scope, duration, and linkage summary](https://www.learncpp.com/cpp-tutorial/scope-duration-and-linkage-summary/).

------

#### Quiz time

**Question #1**

What’s the difference between a variable’s scope, duration, and linkage? What kind of scope, duration, and linkage do global variables have?

Scope determines where a variable is accessible. Duration determines when a variable is created and destroyed. Linkage determines whether the variable can be exported to another file or not.

Global variables have global scope (aka. file scope), which means they can be accessed from the point of declaration to the end of the file in which they are declared.

Global variables have static duration, which means they are created when the program is started, and destroyed when it ends.

Global variables can have either internal or external linkage, via the static and extern keywords respectively.

------

### 6.8 — Why (non-const) global variables are evil

If you were to ask a veteran programmer for *one* piece of advice on good programming practices, after some thought, the most likely answer would be, “Avoid global variables!”. And with good reason: global variables are one of the most historically abused concepts in the language. Although they may seem harmless in small academic programs, they are often problematic in larger ones.

New programmers are often tempted to use lots of global variables, because they are easy to work with, especially when many calls to different functions are involved (passing data through function parameters is a pain). However, this is generally a bad idea. Many developers believe non-const global variables should be avoided completely!

But before we go into why, we should make a clarification. When developers tell you that global variables are evil, they’re usually not talking about *all* global variables. They’re mostly talking about non-const global variables.

------

#### Why (non-const) global variables are evil

By far the biggest reason non-const global variables are dangerous is because their values can be changed by *any* function that is called, and there is no easy way for the programmer to know that this will happen. Consider the following program:

```cpp
#include <iostream>

int g_mode; // declare global variable (will be zero-initialized by default)

void doSomething()
{
    g_mode = 2; // set the global g_mode variable to 2
}

int main()
{
    g_mode = 1; // note: this sets the global g_mode variable to 1.  It does not declare a local g_mode variable!

    doSomething();

    // Programmer still expects g_mode to be 1
    // But doSomething changed it to 2!

    if (g_mode == 1)
    {
        std::cout << "No threat detected.\n";
    }
    else
    {
        std::cout << "Launching nuclear missiles...\n";
    }

    return 0;
}
```

Note that the programmer set variable `g_mode` to *1*, and then called `doSomething()`. Unless the programmer had explicit knowledge that `doSomething()` was going to change the value of `g_mode`, he or she was probably not expecting `doSomething()` to change the value! Consequently, the rest of `main()` doesn’t work like the programmer expects (and the world is obliterated).

In short, global variables make the program’s state unpredictable. Every function call becomes potentially dangerous, and the programmer has no easy way of knowing which ones are dangerous and which ones aren’t! Local variables are much safer because other functions can not affect them directly.

There are plenty of other good reasons not to use non-const globals.

With global variables, it’s not uncommon to find a piece of code that looks like this:

```cpp
void someFunction()
{
    // useful code

    if (g_mode == 4)
    {
        // do something good
    }
}
```

After debugging, you determine that your program isn’t working correctly because `g_mode` has value `3`, not *4*. How do you fix it? Now you need to find all of the places `g_mode` could possibly be set to `3`, and trace through how it got set in the first place. It’s possible this may be in a totally unrelated piece of code!

One of the key reasons to declare local variables as close to where they are used as possible is because doing so minimizes the amount of code you need to look through to understand what the variable does. Global variables are at the opposite end of the spectrum -- because they can be accessed anywhere, you might have to look through the entire program to understand their usage. In small programs, this might not be an issue. In large ones, it will be.

For example, you might find `g_mode` is referenced 442 times in your program. Unless `g_mode` is well documented, you’ll potentially have to look through every use of `g_mode` to understand how it’s being used in different cases, what its valid values are, and what its overall function is.

Global variables also make your program less modular and less flexible. A function that utilizes nothing but its parameters and has no side effects is perfectly modular. Modularity helps both in understanding what a program does, as well as with reusability. Global variables reduce modularity significantly.

In particular, avoid using global variables for important “decision-point” variables (e.g. variables you’d use in a conditional statement, like variable `g_mode` in the example above). Your program isn’t likely to break if a global variable holding an informational value changes (e.g. like the user’s name). It is much more likely to break if you change a global variable that impacts *how* your program actually functions.

**Best practice**

Use local variables instead of global variables whenever possible.

------

#### The initialization order problem of global variables

Initialization of static variables (which includes global variables) happens as part of program startup, before execution of the `main` function. This proceeds in two phases.

The first phase is called `static initialization`. In the static initialization phase, global variables with constexpr initializers (including literals) are initialized to those values. Also, global variables without initializers are zero-initialized.

The second phase is called `dynamic initialization`. This phase is more complex and nuanced, but the gist of it is that global variables with non-constexpr initializers are initialized.

Here’s an example of a non-constexpr initializer:

```cpp
int init()
{
    return 5;
}

int g_something{ init() }; // non-constexpr initialization
```

Within a single file, global variables are generally initialized in order of definition (there are a few exceptions to this rule). Given this, you need to be careful not to have variables dependent on the initialization value of other variables that won’t be initialized until later. For example:

```cpp
#include <iostream>

int initx();  // forward declaration
int inity();  // forward declaration

int g_x{ initx() }; // g_x is initialized first
int g_y{ inity() };

int initx()
{
    return g_y; // g_y isn't initialized when this is called
}

int inity()
{
    return 5;
}

int main()
{
    std::cout << g_x << ' ' << g_y << '\n';
}
```

This prints:

```
0 5
```

Much more of a problem, the order of initialization across different files is not defined. Given two files, `a.cpp` and `b.cpp`, either could have its global variables initialized first. This means that if the variables in `a.cpp` are dependent upon the values in `b.cpp`, there’s a 50% chance that those variables won’t be initialized yet.

**Warning**

Dynamic initialization of global variables causes a lot of problems in C++. Avoid dynamic initialization whenever possible.

------

#### So what are very good reasons to use non-const global variables?

There aren’t many. In most cases, there are other ways to solve the problem that avoids the use of non-const global variables. But in some cases, judicious use of non-const global variables *can* actually reduce program complexity, and in these rare cases, their use may be better than the alternatives.

A good example is a log file, where you can dump error or debug information. It probably makes sense to define this as a global, because you’re likely to only have one log in a program and it will likely be used everywhere in your program.

For what it’s worth, the std::cout and std::cin objects are implemented as global variables (inside the *std* namespace).

As a rule of thumb, any use of a global variable should meet at least the following two criteria: There should only ever be one of the thing the variable represents in your program, and its use should be ubiquitous throughout your program.

Many new programmers make the mistake of thinking that something can be implemented as a global because only one is needed *right now*. For example, you might think that because you’re implementing a single player game, you only need one player. But what happens later when you want to add a multiplayer mode (versus or hotseat)?

------

#### Protecting yourself from global destruction

If you do find a good use for a non-const global variable, a few useful bits of advice will minimize the amount of trouble you can get into. This advice isn’t only for non-const global variables, but can help with all global variables.

First, prefix all non-namespaced global variables with “g” or “g_”, or better yet, put them in a namespace (discussed in lesson [6.2 -- User-defined namespaces and the scope resolution operator](https://www.learncpp.com/cpp-tutorial/user-defined-namespaces-and-the-scope-resolution-operator/)), to reduce the chance of naming collisions.

For example, instead of:

```cpp
constexpr double gravity { 9.8 }; // unclear if this is a local or global variable from the name

int main()
{
    return 0;
}
```

Do this:

```cpp
namespace constants
{
    constexpr double gravity { 9.8 };
}

int main()
{
    return 0;
}
```

Second, instead of allowing direct access to the global variable, it’s a better practice to “encapsulate” the variable. Make sure the variable can only be accessed from within the file it’s declared in, e.g. by making the variable static or const, then provide external global “access functions” to work with the variable. These functions can ensure proper usage is maintained (e.g. do input validation, range checking, etc…). Also, if you ever decide to change the underlying implementation (e.g. move from one database to another), you only have to update the access functions instead of every piece of code that uses the global variable directly.

For example, instead of:

```cpp
namespace constants
{
    extern const double gravity { 9.8 }; // has external linkage, is directly accessible by other files
}
```

Do this:

```cpp
namespace constants
{
    constexpr double gravity { 9.8 }; // has internal linkage, is accessible only by this file
}

double getGravity() // this function can be exported to other files to access the global outside of this file
{
    // We could add logic here if needed later
    // or change the implementation transparently to the callers
    return constants::gravity;
}
```

**A reminder**

Global `const` variables have internal linkage by default, `gravity` doesn’t need to be `static`.

Third, when writing an otherwise standalone function that uses the global variable, don’t use the variable directly in your function body. Pass it in as an argument instead. That way, if your function ever needs to use a different value for some circumstance, you can simply vary the argument. This helps maintain modularity.

Instead of:

```cpp
#include <iostream>

namespace constants
{
    constexpr double gravity { 9.8 };
}

// This function is only useful for calculating your instant velocity based on the global gravity
double instantVelocity(int time)
{
    return constants::gravity * time;
}

int main()
{
    std::cout << instantVelocity(5);
}
```

Do this:

```cpp
#include <iostream>

namespace constants
{
    constexpr double gravity { 9.8 };
}

// This function can calculate the instant velocity for any gravity value (more useful)
double instantVelocity(int time, double gravity)
{
    return gravity * time;
}

int main()
{
    std::cout << instantVelocity(5, constants::gravity); // pass our constant to the function as a parameter
}
```

**A joke**

What’s the best naming prefix for a global variable?

Answer: //

C++ jokes are the best.

------

### 6.9 — Sharing global constants across multiple files (using inline variables)

In some applications, certain symbolic constants may need to be used throughout your code (not just in one location). These can include physics or mathematical constants that don’t change (e.g. pi or Avogadro’s number), or application-specific “tuning” values (e.g. friction or gravity coefficients). Instead of redefining these constants in every file that needs them (a violation of the “Don’t Repeat Yourself” rule), it’s better to declare them once in a central location and use them wherever needed. That way, if you ever need to change them, you only need to change them in one place, and those changes can be propagated out.

This lesson discusses the most common ways to do this.

------

#### Global constants as internal variables

Prior to C++17, the following is the easiest and most common solution:

1. Create a header file to hold these constants
2. Inside this header file, define a namespace (discussed in lesson [6.2 -- User-defined namespaces and the scope resolution operator](https://www.learncpp.com/cpp-tutorial/user-defined-namespaces-and-the-scope-resolution-operator/))
3. Add all your constants inside the namespace (make sure they’re *constexpr*)
4. \#include the header file wherever you need it

For example:

constants.h:

```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

// define your own namespace to hold constants
namespace constants
{
    // constants have internal linkage by default
    constexpr double pi { 3.14159 };
    constexpr double avogadro { 6.0221413e23 };
    constexpr double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
    // ... other related constants
}
#endif
```

Then use the scope resolution operator (::) with the namespace name to the left, and your variable name to the right in order to access your constants in .cpp files:

main.cpp:

```cpp
#include "constants.h" // include a copy of each constant in this file

#include <iostream>

int main()
{
    std::cout << "Enter a radius: ";
    int radius{};
    std::cin >> radius;

    std::cout << "The circumference is: " << 2 * radius * constants::pi << '\n';

    return 0;
}
```

When this header gets #included into a .cpp file, each of these variables defined in the header will be copied into that code file at the point of inclusion. Because these variables live outside of a function, they’re treated as global variables within the file they are included into, which is why you can use them anywhere in that file.

Because const globals have internal linkage, each .cpp file gets an independent version of the global variable that the linker can’t see. In most cases, because these are const, the compiler will simply optimize the variables away.

**As an aside…**

The term “optimizing away” refers to any process where the compiler optimizes the performance of your program by removing things in a way that doesn’t affect the output of your program. For example, lets say you have some const variable `x` that’s initialized to value `4`. Wherever your code references variable `x`, the compiler can just replace `x` with `4` (since `x` is const, we know it won’t ever change to a different value) and avoid having to create and initialize a variable altogether.

------

#### Global constants as external variables

The above method has a few potential downsides.

While this is simple (and fine for smaller programs), every time constants.h gets #included into a different code file, each of these variables is copied into the including code file. Therefore, if constants.h gets included into 20 different code files, each of these variables is duplicated 20 times. Header guards won’t stop this from happening, as they only prevent a header from being included more than once into a single including file, not from being included one time into multiple different code files. This introduces two challenges:

1. Changing a single constant value would require recompiling every file that includes the constants header, which can lead to lengthy rebuild times for larger projects.
2. If the constants are large in size and can’t be optimized away, this can use a lot of memory.

One way to avoid these problems is by turning these constants into external variables, since we can then have a single variable (initialized once) that is shared across all files. In this method, we’ll define the constants in a .cpp file (to ensure the definitions only exist in one place), and put forward declarations in the header (which will be included by other files).

**Author’s note**

We use const instead of constexpr in this method because constexpr variables can’t be forward declared, even if they have external linkage. This is because the compiler needs to know the value of the variable at compile time, and a forward declaration does not provide this information.

constants.cpp:

```cpp
#include "constants.h"

namespace constants
{
    // actual global variables
    extern const double pi { 3.14159 };
    extern const double avogadro { 6.0221413e23 };
    extern const double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
}
```

constants.h:

```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace constants
{
    // since the actual variables are inside a namespace, the forward declarations need to be inside a namespace as well
    extern const double pi;
    extern const double avogadro;
    extern const double myGravity;
}

#endif
```

Use in the code file stays the same:

main.cpp:

```cpp
#include "constants.h" // include all the forward declarations

#include <iostream>

int main()
{
    std::cout << "Enter a radius: ";
    int radius{};
    std::cin >> radius;

    std::cout << "The circumference is: " << 2 * radius * constants::pi << '\n';

    return 0;
}
```

Because global symbolic constants should be namespaced (to avoid naming conflicts with other identifiers in the global namespace), the use of a “g_” naming prefix is not necessary.

Now the symbolic constants will get instantiated only once (in `constants.cpp`) instead of in each code file where `constants.h` is #included, and all uses of these constants will be linked to the version instantiated in `constants.cpp`. Any changes made to `constants.cpp` will require recompiling only `constants.cpp`.

However, there are a couple of downsides to this method. First, these constants are now considered compile-time constants only within the file they are actually defined in (`constants.cpp`). In other files, the compiler will only see the forward declaration, which doesn’t define a constant value (and must be resolved by the linker). This means in other files, these are treated as runtime constant values, not compile-time constants. Thus outside of `constants.cpp`, these variables can’t be used anywhere that requires a compile-time constant. Second, because compile-time constants can typically be optimized more than runtime constants, the compiler may not be able to optimize these as much.

**Key insight**

In order for variables to be usable in compile-time contexts, such as array sizes, the compiler has to see the variable’s definition (not just a forward declaration).

Because the compiler compiles each source file individually, it can only see variable definitions that appear in the source file being compiled (which includes any included headers). For example, variable definitions in `constants.cpp` are not visible when the compiler compiles `main.cpp`. For this reason, `constexpr` variables cannot be separated into header and source file, they have to be defined in the header file.

Given the above downsides, prefer defining your constants in a header file (either per the prior section, or per the next section). If you find that the values for your constants are changing a lot (e.g. because you are tuning the program) and this is leading to long compilation times, you can move just the offending constants into a .cpp file as needed.

------

#### Global constants as inline variables C++17

C++17 introduced a new concept called `inline variables`. In C++, the term `inline` has evolved to mean “multiple definitions are allowed”. Thus, an **inline variable** is one that is allowed to be defined in multiple files without violating the one definition rule. Inline global variables have external linkage by default.

The linker will consolidate all inline definitions of a variable into a single variable definition (thus meeting the one definition rule). This allows us to define variables in a header file and have them treated as if there was only one definition in a .cpp file somewhere. Let’s say you have a normal constant that you’re #including into 10 code files. Without inline, you get 10 definitions. With inline, the compiler picks 1 definition to be the canonical definition, so you only get 1 definition. This means you save 9 constants worth of memory.

These variables will also retain their constexpr-ness in all files in which they are included, so they can be used anywhere a constexpr value is required. Constexpr values can also be more highly optimized by the compiler than runtime-const (or non-const) variables.

Inline variables have two primary restrictions that must be obeyed:

1. All definitions of the inline variable must be identical (otherwise, undefined behavior will result).
2. The inline variable definition (not a forward declaration) must be present in any file that uses the variable.

With this, we can go back to defining our globals in a header file without the downside of duplicated variables:

constants.h:

```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

// define your own namespace to hold constants
namespace constants
{
    inline constexpr double pi { 3.14159 }; // note: now inline constexpr
    inline constexpr double avogadro { 6.0221413e23 };
    inline constexpr double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
    // ... other related constants
}
#endif
```

main.cpp:

### #include "constants.h"


#include <iostream>

int main()

{

    std::cout << "Enter a radius: ";
    
    int radius{};
    
    std::cin >> radius;


    std::cout << "The circumference is: " << 2 * radius * constants::pi << '\n';


    return 0;

}

We can include `constants.h` into as many code files as we want, but these variables will only be instantiated once and shared across all code files.

This method does retain the downside of requiring every file that includes the constants header be recompiled if any constant value is changed.

**Best practice**

If you need global constants and your compiler is C++17 capable, prefer defining inline constexpr global variables in a header file.

**A reminder**

Use `std::string_view` for `constexpr` strings. We cover this in lesson [4.18 -- Introduction to std::string_view](https://www.learncpp.com/cpp-tutorial/introduction-to-stdstring_view/).

------

### 6.10 — Static local variables

The term `static` is one of the most confusing terms in the C++ language, in large part because `static` has different meanings in different contexts.

In prior lessons, we covered that global variables have `static duration`, which means they are created when the program starts and destroyed when the program ends.

We also discussed how the `static` keyword gives a global identifier `internal linkage`, which means the identifier can only be used in the file in which it is defined.

In this lesson, we’ll explore the use of the `static` keyword when applied to a local variable.

------

#### Static local variables

In lesson [2.5 -- Introduction to local scope](https://www.learncpp.com/cpp-tutorial/introduction-to-local-scope/), you learned that local variables have `automatic duration` by default, which means they are created at the point of definition, and destroyed when the block is exited.

Using the `static` keyword on a local variable changes its duration from `automatic duration` to `static duration`. This means the variable is now created at the start of the program, and destroyed at the end of the program (just like a global variable). As a result, the static variable will retain its value even after it goes out of scope!

The easiest way to show the difference between `automatic duration` and `static duration` variables is by example.

Automatic duration (default):

```cpp
#include <iostream>

void incrementAndPrint()
{
    int value{ 1 }; // automatic duration by default
    ++value;
    std::cout << value << '\n';
} // value is destroyed here

int main()
{
    incrementAndPrint();
    incrementAndPrint();
    incrementAndPrint();

    return 0;
}
```

Each time incrementAndPrint() is called, a variable named value is created and assigned the value of 1. incrementAndPrint() increments value to 2, and then prints the value of 2. When incrementAndPrint() is finished running, the variable goes out of scope and is destroyed. Consequently, this program outputs:

```
2
2
2
```

Now consider the static version of this program. The only difference between this and the above program is that we’ve changed the local variable from `automatic duration` to `static duration` by using the `static` keyword.

Static duration (using static keyword):

```cpp
#include <iostream>

void incrementAndPrint()
{
    static int s_value{ 1 }; // static duration via static keyword.  This initializer is only executed once.
    ++s_value;
    std::cout << s_value << '\n';
} // s_value is not destroyed here, but becomes inaccessible because it goes out of scope

int main()
{
    incrementAndPrint();
    incrementAndPrint();
    incrementAndPrint();

    return 0;
}
```

In this program, because `s_value` has been declared as `static`, it is created at the program start.

Static local variables that are zero initialized or have a constexpr initializer can be initialized at program start.

Static local variables that have no initializer or a non-constexpr initializer are zero-initialized at program start. Static local variables with a non-constexpr initializer are reinitialized the first time the variable definition is encountered. The definition is skipped on subsequent calls, so no futher reinitialization happens. Because they have static duration, static local variables that are not explicitly initialized will be zero-initialized by default.

Because `s_value` has constexpr initializer `1`, `s_value` will be initialized at program start.

When `s_value` goes out of scope at the end of the function, it is not destroyed. Each time the function incrementAndPrint() is called, the value of `s_value` remains at whatever we left it at previously. Consequently, this program outputs:

```
2
3
4
```

Just like we use “g_” to prefix global variables, it’s common to use “s_” to prefix static (static duration) local variables.

One of the most common uses for static duration local variables is for unique ID generators. Imagine a program where you have many similar objects (e.g. a game where you’re being attacked by many zombies, or a simulation where you’re displaying many triangles). If you notice a defect, it can be near impossible to distinguish which object is having problems. However, if each object is given a unique identifier upon creation, then it can be easier to differentiate the objects for further debugging.

Generating a unique ID number is very easy to do with a static duration local variable:

```cpp
int generateID()
{
    static int s_itemID{ 0 };
    return s_itemID++; // makes copy of s_itemID, increments the real s_itemID, then returns the value in the copy
}
```

The first time this function is called, it returns 0. The second time, it returns 1. Each time it is called, it returns a number one higher than the previous time it was called. You can assign these numbers as unique IDs for your objects. Because `s_itemID` is a local variable, it can not be “tampered with” by other functions.

Static variables offer some of the benefit of global variables (they don’t get destroyed until the end of the program) while limiting their visibility to block scope. This makes them safer for use even if you change their values regularly.

**Best practice**

Initialize your static local variables. Static local variables are only initialized the first time the code is executed, not on subsequent calls.

------

#### Static local constants

Static local variables can be made const (or constexpr). One good use for a const static local variable is when you have a function that needs to use a const value, but creating or initializing the object is expensive (e.g. you need to read the value from a database). If you used a normal local variable, the variable would be created and initialized every time the function was executed. With a const/constexpr static local variable, you can create and initialize the expensive object once, and then reuse it whenever the function is called.

------

#### Don’t use static local variables to alter flow

Consider the following code:

```cpp
#include <iostream>

int getInteger()
{
	static bool s_isFirstCall{ true };

	if (s_isFirstCall)
	{
		std::cout << "Enter an integer: ";
		s_isFirstCall = false;
	}
	else
	{
		std::cout << "Enter another integer: ";
	}

	int i{};
	std::cin >> i;
	return i;
}

int main()
{
	int a{ getInteger() };
	int b{ getInteger() };

	std::cout << a << " + " << b << " = " << (a + b) << '\n';

	return 0;
}
```

Sample output

```
Enter an integer: 5
Enter another integer: 9
5 + 9 = 14
```

This code does what it’s supposed to do, but because we used a static local variable, we made the code harder to understand. If someone reads the code in `main()` without reading the implementation of `getInteger()`, they’d have no reason to assume that the two calls to `getInteger()` do something different. But the two calls do something different, which can be very confusing if the difference is more than a changed prompt.

Say you press the +1 button on your microwave and the microwave adds 1 minute to the remaining time. Your meal is warm and you’re happy. Before you take your meal out of the microwave, you see a cat outside your window and watch it for a moment, because cats are cool. The moment turned out to be longer than you expected and when you take the first bite of your meal, it’s cold again. No problem, just put it back into the microwave and press +1 to run it for a minute. But this time the microwave adds only 1 second and not 1 minute. That’s when you go “I changed nothing and now it’s broken” or “It worked last time”. If you do the same thing again, you’d expect the same behavior as last time. The same goes for functions.

Suppose we want to add subtraction to the calculator such that the output looks like the following:

```
Addition
Enter an integer: 5
Enter another integer: 9
5 + 9 = 14
Subtraction
Enter an integer: 12
Enter another integer: 3
12 - 3 = 9
```

We might try to use `getInteger()` to read in the next two integers like we did for addition.

```cpp
int main()
{
  std::cout << "Addition\n";

  int a{ getInteger() };
  int b{ getInteger() };

  std::cout << a << " + " << b << " = " << (a + b) << '\n';

  std::cout << "Subtraction\n";

  int c{ getInteger() };
  int d{ getInteger() };

  std::cout << c << " - " << d << " = " << (c - d) << '\n';

  return 0;
}
```

But this won’t work, the output is

```
Addition
Enter an integer: 5
Enter another integer: 9
5 + 9 = 14
Subtraction
Enter another integer: 12
Enter another integer: 3
12 - 3 = 9
```

(“Enter another integer” instead of “Enter an integer”)

`getInteger()` is not reusable, because it has an internal state (The static local variable `s_isFirstCall`) which cannot be reset from the outside. `s_isFirstCall` is not a variable that should be unique in the entire program. Although our program worked great when we first wrote it, the static local variable prevents us from reusing the function later on.

A better way of implementing `getInteger` is to pass `s_isFirstCall` as a parameter. This allows the caller to choose which prompt will be printed.

Static local variables should only be used if in your entire program and in the foreseeable future of your program, the variable is unique and it wouldn’t make sense to reset the variable.

**Best practice**

Avoid `static` local variables unless the variable never needs to be reset.

------

#### Quiz time

**Question #1**

What effect does using keyword `static` have on a global variable? What effect does it have on a local variable?

When applied to a global variable, the static keyword defines the global variable as having internal linkage, meaning the variable cannot be exported to other files.

When applied to a local variable, the static keyword defines the local variable as having static duration, meaning the variable will only be created once, and will not be destroyed until the end of the program.

------

### 6.11 — Scope, duration, and linkage summary

The concepts of scope, duration, and linkage cause a lot of confusion, so we’re going to take an extra lesson to summarize everything. Some of these things we haven’t covered yet, and they’re here just for completeness / reference later.

------

#### Scope summary

An identifier’s *scope* determines where the identifier can be accessed within the source code.

- Variables with **block (local) scope** can only be accessed from the point of declaration until the end of the block in which they are declared (including nested blocks). This includes:
  - Local variables
  - Function parameters
  - User-defined type definitions (such as enums and classes) declared inside a block
- Variables and functions with **file (global) scope** can be accessed from the point of declaration until the end of the file. This includes:
  - Global variables
  - Functions
  - User-defined type definitions (such as enums and classes) declared inside a namespace or in the global scope

------

#### Duration summary

A variable’s *duration* determines when it is created and destroyed.

- Variables with **automatic duration** are created at the point of definition, and destroyed when the block they are part of is exited. This includes:
  - Local variables
  - Function parameters
- Variables with **static duration** are created when the program begins and destroyed when the program ends. This includes:
  - Global variables
  - Static local variables
- Variables with **dynamic duration** are created and destroyed by programmer request. This includes:
  - Dynamically allocated variables

------

#### Linkage summary

An identifier’s *linkage* determines whether multiple declarations of an identifier refer to the same entity (object, function, reference, etc…) or not.

- An identifier with **no linkage** means the identifier only refers to itself. This includes:
  - Local variables
  - User-defined type definitions (such as enums and classes) declared inside a block
- An identifier with **internal linkage** can be accessed anywhere within the file it is declared. This includes:
  - Static global variables (initialized or uninitialized)
  - Static functions
  - Const global variables
  - Functions declared inside an unnamed namespace
  - User-defined type definitions (such as enums and classes) declared inside an unnamed namespace
- An identifier with **external linkage** can be accessed anywhere within the file it is declared, or other files (via a forward declaration). This includes:
  - Functions
  - Non-const global variables (initialized or uninitialized)
  - Extern const global variables
  - Inline const global variables
  - User-defined type definitions (such as enums and classes) declared inside a namespace or in the global scope

Identifiers with external linkage will generally cause a duplicate definition linker error if the definitions are compiled into more than one .cpp file (due to violating the one-definition rule). There are some exceptions to this rule (for types, templates, and inline functions and variables) -- we’ll cover these further in future lessons when we talk about those topics.

Also note that functions have external linkage by default. They can be made internal by using the static keyword.

------

#### Variable scope, duration, and linkage summary

Because variables have scope, duration, and linkage, let’s summarize in a chart:

| Type                                    | Example                         | Scope | Duration  | Linkage  | Notes                        |
| :-------------------------------------- | :------------------------------ | :---- | :-------- | :------- | :--------------------------- |
| Local variable                          | int x;                          | Block | Automatic | None     |                              |
| Static local variable                   | static int s_x;                 | Block | Static    | None     |                              |
| Dynamic variable                        | int* x { new int{} };           | Block | Dynamic   | None     |                              |
| Function parameter                      | void foo(int x)                 | Block | Automatic | None     |                              |
| External non-constant global variable   | int g_x;                        | File  | Static    | External | Initialized or uninitialized |
| Internal non-constant global variable   | static int g_x;                 | File  | Static    | Internal | Initialized or uninitialized |
| Internal constant global variable       | constexpr int g_x { 1 };        | File  | Static    | Internal | Must be initialized          |
| External constant global variable       | extern const int g_x { 1 };     | File  | Static    | External | Must be initialized          |
| Inline constant global variable (C++17) | inline constexpr int g_x { 1 }; | File  | Static    | External | Must be initialized          |

------

#### Forward declaration summary

You can use a forward declaration to access a function or variable in another file. The scope of the declared variable is as per usual (file scope for globals, block scope for locals).

| Type                                      | Example                   | Notes                                             |
| :---------------------------------------- | :------------------------ | :------------------------------------------------ |
| Function forward declaration              | void foo(int x);          | Prototype only, no function body                  |
| Non-constant variable forward declaration | extern int g_x;           | Must be uninitialized                             |
| Const variable forward declaration        | extern const int g_x;     | Must be uninitialized                             |
| Constexpr variable forward declaration    | extern constexpr int g_x; | Not allowed, constexpr cannot be forward declared |

------

#### What the heck is a storage class specifier?

When used as part of an identifier declaration, the `static` and `extern` keywords are called **storage class specifiers**. In this context, they set the storage duration and linkage of the identifier.

C++ supports 4 active storage class specifiers:

| Specifier    | Meaning                                                      | Note                |
| :----------- | :----------------------------------------------------------- | :------------------ |
| extern       | static (or thread_local) storage duration and external linkage |                     |
| static       | static (or thread_local) storage duration and internal linkage |                     |
| thread_local | thread storage duration                                      |                     |
| mutable      | object allowed to be modified even if containing class is const |                     |
| auto         | automatic storage duration                                   | Deprecated in C++11 |
| register     | automatic storage duration and hint to the compiler to place in a register | Deprecated in C++17 |

The term *storage class specifier* is typically only used in formal documentation.

------

### 6.12 — Using declarations and using directives

You’ve probably seen this program in a lot of textbooks and tutorials:

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout << "Hello world!\n";

    return 0;
}
```

Some older IDEs will also auto-populate new C++ projects with a similar program (so you can compile something immediately, rather than starting from a blank file).

If you see this, run. Your textbook, tutorial, or compiler are probably out of date. In this lesson, we’ll explore why.

------

#### A short history lesson

Back before C++ had support for namespaces, all of the names that are now in the `std` namespace were in the global namespace. This caused naming collisions between program identifiers and standard library identifiers. Programs that worked under one version of C++ might have a naming conflict with a newer version of C++.

In 1995, namespaces were standardized, and all of the functionality from the standard library was moved out of the global namespace and into namespace `std`. This change broke older code that was still using names without `std::`.

As anyone who has worked on a large codebase knows, any change to a codebase (no matter how trivial) risks breaking the program. Updating every name that was now moved into the `std` namespace to use the `std::` prefix was a massive risk. A solution was requested.

Fast forward to today -- if you’re using the standard library a lot, typing `std::` before everything you use from the standard library can become repetitive, and in some cases, can make your code harder to read.

C++ provides some solutions to both of these problems, in the form of `using statements`.

But first, let’s define two terms.

------

#### Qualified and unqualified names

A name can be either qualified or unqualified.

A **qualified name** is a name that includes an associated scope. Most often, names are qualified with a namespace using the scope resolution operator (::). For example:

```cpp
std::cout // identifier cout is qualified by namespace std
::foo // identifier foo is qualified by the global namespace
```

**For advanced readers**

A name can also be qualified by a class name using the scope resolution operator (::), or by a class object using the member selection operators (. or ->). For example:

```cpp
class C; // some class

C::s_member; // s_member is qualified by class C
obj.x; // x is qualified by class object obj
ptr->y; // y is qualified by pointer to class object ptr
```

An **unqualified name** is a name that does not include a scoping qualifier. For example, `cout` and `x` are unqualified names, as they do not include an associated scope.

------

#### Using declarations

One way to reduce the repetition of typing `std::` over and over is to utilize a `using declaration` statement. A **using declaration** allows us to use an unqualified name (with no scope) as an alias for a qualified name.

Here’s our basic Hello world program, using a `using declaration` on line 5:

```cpp
#include <iostream>

int main()
{
   using std::cout; // this using declaration tells the compiler that cout should resolve to std::cout
   cout << "Hello world!\n"; // so no std:: prefix is needed here!

   return 0;
} // the using declaration expires here
```

The `using declaration` `using std::cout;` tells the compiler that we’re going to be using the object `cout` from the `std namespace`. So whenever it sees `cout`, it will assume that we mean `std::cout`. If there’s a naming conflict between `std::cout` and some other use of `cout`, `std::cout` will be preferred. Therefore on line 6, we can type `cout` instead of `std::cout`.

This doesn’t save much effort in this trivial example, but if you are using `cout` many times inside of a function, a `using declaration` can make your code more readable. Note that you will need a separate `using declaration` for each name (e.g. one for `std::cout`, one for `std::cin`, etc…).

Although this method is less explicit than using the `std::` prefix, it’s generally considered safe and acceptable (when used inside a function).

------

#### Using directives

Another way to simplify things is to use a `using directive`. Slightly simplified, a **using directive** imports all of the identifiers from a namespace into the scope of the `using directive`.

**For advanced readers**

For technical reasons, using directives do not actually import names into the current scope -- instead they import the names into an outer scope (more details about which outer scope is picked can be found [here](https://quuxplusone.github.io/blog/2020/12/21/using-directive/). However, these names are not accessible from the outer scope -- they are *only* accessible via unqualified (non-prefixed) lookup from the scope of the using directive (or a nested scope).

The practical effect is that (outside of some weird edge cases involving multiple using directives inside nested namespaces), using directives behave as if the names had been imported into the current scope. To keep things simple, we will proceed under the simplification that the names are imported into the current scope.

Here’s our Hello world program again, with a `using directive` on line 5:

```cpp
#include <iostream>

int main()
{
   using namespace std; // this using directive tells the compiler to import all names from namespace std into the current namespace without qualification
   cout << "Hello world!\n"; // so no std:: prefix is needed here
   return 0;
}
```

The `using directive` `using namespace std;` tells the compiler to import *all* of the names from the `std namespace` into the current scope (in this case, of function `main()`). When we then use unqualified identifier `cout`, it will resolve to the imported `std::cout`.

`Using directives` are the solution that was provided for old pre-namespace codebases that used unqualified names for standard library functionality. Rather than having to manually update every unqualified name to a qualified name (which was risky), a single `using directive` (of `using namespace std;`) could be placed at the top of the each file, and all of the names that had been moved to the `std` namespace could still be used unqualified.

------

#### Problems with using directives (a.k.a. why you should avoid “using namespace std;”) 

In modern C++, `using directives` generally offer little benefit (saving some typing) compared to the risk. Because using directives import *all* of the names from a namespace (potentially including lots of names you’ll never use), the possibility for naming collisions to occur increases significantly (especially if you import the `std` namespace).

For illustrative purposes, let’s take a look at an example where `using directives` cause ambiguity:

```cpp
#include <iostream>

namespace a
{
	int x{ 10 };
}

namespace b
{
	int x{ 20 };
}

int main()
{
	using namespace a;
	using namespace b;

	std::cout << x << '\n';

	return 0;
}
```

In the above example, the compiler is unable to determine whether the `x` in `main` refers to `a::x` or `b::x`. In this case, it will fail to compile with an “ambiguous symbol” error. We could resolve this by removing one of the `using` statements, employing a `using declaration` instead, or qualifying `x` with an explicit scope qualifier (`a::` or `b::`).

Here’s another more subtle example:

```cpp
#include <iostream> // imports the declaration of std::cout

int cout() // declares our own "cout" function
{
    return 5;
}

int main()
{
    using namespace std; // makes std::cout accessible as "cout"
    cout << "Hello, world!\n"; // uh oh!  Which cout do we want here?  The one in the std namespace or the one we defined above?

    return 0;
}
```

In the above example, the compiler is unable to determine whether our use of `cout` means `std::cout` or the `cout` function we’ve defined, and again will fail to compile with an “ambiguous symbol” error. Although this example is trivial, if we had explicitly prefixed `std::cout` like this:

```cpp
std::cout << "Hello, world!\n"; // tell the compiler we mean std::cout
```

or used a `using declaration` instead of a `using directive`:

```cpp
using std::cout; // tell the compiler that cout means std::cout
cout << "Hello, world!\n"; // so this means std::cout
```

then our program wouldn’t have any issues in the first place. And while you’re probably not likely to write a function named “cout”, there are hundreds, if not thousands, of other names in the std namespace just waiting to collide with your names. “count”, “min”, “max”, “search”, “sort”, just to name a few.

Even if a `using directive` does not cause naming collisions today, it makes your code more vulnerable to future collisions. For example, if your code includes a `using directive` for some library that is then updated, all of the new names introduced in the updated library are now candidates for naming collisions with your existing code.

There is a more insidious problem that can occur as well. The updated library may introduce a function that not only has the same name, but is actually a better match for some function call. In such a case, the compiler may decide to prefer the new function instead, and the behavior of your program will change unexpectedly.

Consider the following program:

foolib.h (part of some third-party library):

```cpp
#ifndef FOOLIB_H
#define FOOLIB_H

namespace foo
{
    // pretend there is some useful code that we use here
}
#endif
```

main.cpp:

```cpp
#include <iostream>
#include <foolib.h> // a third-party library, thus angled brackets used

int someFcn(double)
{
    return 1;
}

int main()
{
    using namespace foo; // Because we're lazy and want to access foo:: qualified names without typing the foo:: prefix
    std::cout << someFcn(0) << '\n'; // The literal 0 should be 0.0, but this is an easy mistake to make

    return 0;
}
```

This program runs and prints `1`.

Now, let’s say we update the foolib library, which includes an updated foolib.h. Our program now looks like this:

foolib.h (part of some third-party library):

```cpp
#ifndef FOOLIB_H
#define FOOLIB_H

namespace foo
{
    // newly introduced function
    int someFcn(int)
    {
        return 2;
    }

    // pretend there is some useful code that we use here
}
#endif
```

main.cpp:

```cpp
#include <iostream>
#include <foolib.h>

int someFcn(double)
{
    return 1;
}

int main()
{
    using namespace foo; // Because we're lazy and want to access foo:: qualified names without typing the foo:: prefix
    std::cout << someFcn(0) << '\n'; // The literal 0 should be 0.0, but this is an easy mistake to make

    return 0;
}
```

Our `main.cpp` file hasn’t changed at all, but this program now runs and prints `2`!

When the compiler encounters a function call, it has to determine what function definition it should match the function call with. In selecting a function from a set of potentially matching functions, it will prefer a function that requires no argument conversions over a function that requires argument conversions. Because the literal `0` is an integer, C++ will prefer to match `someFcn(0)` with the newly introduced `someFcn(int)` (no conversions) over `someFcn(double)` (requires a conversion from int to double). That causes an unexpected change to our program results.

This would not have happened if we’d used a `using declaration` or explicit scope qualifier.

Finally, the lack of explicit scope prefixes make it harder for a reader to tell what functions are part of a library and what’s part of your program. For example, if we use a using directive:

```cpp
using namespace ns;

int main()
{
    foo(); // is this foo a user-defined function, or part of the ns library?
}
```

It’s unclear whether the call to `foo()` is actually a call to `ns::foo()` or to a `foo()` that is a user-defined function. Modern IDEs should be able to disambiguate this for you when you hover over a name, but having to hover over each name just to see where it comes from is tedious.

Without the using directive, it’s much clearer:

```cpp
int main()
{
    ns::foo(); // clearly part of the ns library
    foo(); // likely a user-defined function
}
```

In this version, the call to `ns::foo()` is clearly a library call. The call to plain `foo()` is probably a call to a user-defined function (some libraries, including certain standard library headers, do put names into the global namespace, so it’s not a guarantee).

------

#### The scope of using declarations and directives

If a `using declaration` or `using directive` is used within a block, the names are applicable to just that block (it follows normal block scoping rules). This is a good thing, as it reduces the chances for naming collisions to occur to just within that block.

If a `using declaration` or `using directive` is used in the global namespace, the names are applicable to the entire rest of the file (they have file scope).

------

#### Cancelling or replacing a using statement

Once a `using statement` has been declared, there’s no way to cancel or replace it with a different `using statement` within the scope in which it was declared.

```cpp
int main()
{
    using namespace foo;

    // there's no way to cancel the "using namespace foo" here!
    // there's also no way to replace "using namespace foo" with a different using statement

    return 0;
} // using namespace foo ends here
```

The best you can do is intentionally limit the scope of the `using statement` from the outset using the block scoping rules.

```cpp
int main()
{
    {
        using namespace foo;
        // calls to foo:: stuff here
    } // using namespace foo expires

    {
        using namespace Goo;
        // calls to Goo:: stuff here
    } // using namespace Goo expires

    return 0;
}
```

Of course, all of this headache can be avoided by explicitly using the scope resolution operator (::) in the first place.

------

#### Best practices for using statements

Avoid `using directives` (particularly `using namespace std;`), except in specific circumstances (such as `using namespace std::literals` to access the `s` and `sv` literal suffixes). `Using declarations` are generally considered safe to use inside blocks. Limit their use in the global namespace of a code file, and never use them in the global namespace of a header file.

**Best practice**

Prefer explicit namespaces over `using statements`. Avoid `using directives` whenever possible. `Using declarations` are okay to use inside blocks.

**Related content**

The `using` keyword is also used to define type aliases, which are unrelated to using statements. We cover type aliases in lesson [8.7 -- Typedefs and type aliases](https://www.learncpp.com/cpp-tutorial/typedefs-and-type-aliases/).

------

### 6.13 — Inline functions

Consider the case where you need to write some code to perform some discrete task, like reading input from the user, or outputting something to a file, or calculating a particular value. When implementing this code, you essentially have two options:

1. Write the code as part of an existing function (called writing code “in-place” or “inline”).
2. Create a function (and possibly sub-functions) to handle the task.

Writing functions provides many potential benefits, as code in a function:

- Is easier to read and understand in the context of the overall program.
- Is easier to use, as you can call the function without understanding how it is implemented.
- Is easier to update, as the code in a function can be updated in one place.
- Is easier to reuse, as functions are naturally modular.

However, one downside of using a function is that every time a function is called, there is a certain amount of performance overhead that occurs. Consider the following example:

```cpp
#include <iostream>

int min(int x, int y)
{
    return (x < y) ? x : y;
}

int main()
{
    std::cout << min(5, 6) << '\n';
    std::cout << min(3, 2) << '\n';
    return 0;
}
```

When a call to `min()` is encountered, the CPU must store the address of the current instruction it is executing (so it knows where to return to later) along with the values of various CPU registers (so they can be restored upon returning). Then parameters `x` and `y` must be instantiated and then initialized. Then the execution path has to jump to the code in the `min()` function. When the function ends, the program has to jump back to the location of the function call, and the return value has to be copied so it can be output. In other words, there is a significant amount of overhead cost that is incurred with each function call.

For functions that are large and/or perform complex tasks, the overhead of the function call is typically insignificant compared to the amount of time the function takes to run. However, for small functions (such as `min()` above), the overhead costs can be larger than the time needed to actually execute the function’s code! In cases where a small function is called often, using a function can result in a significant performance penalty over writing the same code in-place.

------

#### Inline expansion

Fortunately, the C++ compiler has a trick that it can use to avoid such overhead cost: **Inline expansion** is a process where a function call is replaced by the code from the called function’s definition.

For example, if the compiler expanded the `min()` calls in the above example, the resulting code would look like this:

```cpp
#include <iostream>

int main()
{
    std::cout << ((5 < 6) ? 5 : 6) << '\n';
    std::cout << ((3 < 2) ? 3 : 2) << '\n';
    return 0;
}
```

Note that the two calls to function `min()` have been replaced by the code in the body of the `min()` function (with the value of the arguments substituted for the parameters). This allows us to avoid the overhead of those calls, while preserving the results of the code.

------

#### The performance of inline code

Beyond removing the cost of function call overhead, inline expansion can also allow the compiler to optimize the resulting code more efficiently -- for example, because the expression `((5 < 6) ? 5 : 6)` is now a compile-time constant, the compiler could further optimize the first statement in `main()` to `std::cout << 5 << '\n';`.

However, inline expansion has its own potential cost: if the body of the function being expanded takes more instructions than the function call being replaced, then each inline expansion will cause the executable to grow larger. Larger executables tend to be slower (due to not fitting as well in caches).

The decision about whether a function would benefit from being made inline (because removal of the function call overhead outweighs the cost of a larger executable) is not straightforward. Inline expansion could result in performance improvements, performance reductions, or no change to performance at all, depending on the relative cost of a function call, the size of the function, and what other optimizations can be performed.

Inline expansion is best suited to simple, short functions (e.g. no more than a few statements), especially cases where a single function call is executed more than once (e.g. function calls inside a loop).

------

#### When inline expansion occurs

Every function falls into one of three categories, where calls to the function:

- Must be expanded.
- May be expanded (most functions are in this category).
- Can’t be expanded.

A function that is eligible to have its function calls expanded is called an **inline function**.

Most functions fall into the “may” category: their function calls can be expanded if and when it is beneficial to do so. For functions in this category, a modern compiler will assess each function and each function call to make a determination about whether that particular function call would benefit from inline expansion. A compiler might decide to expand none, some, or all of the function calls to a given function.

**Tip**

Modern optimizing compilers make the decision about when functions should be expanded inline.

**For advanced readers**

Some types of functions are implicitly treated as inline functions. These include:

- Functions defined inside a class, struct, or union type definition.
- Constexpr / consteval functions (6.14 -- Constexpr and consteval functions)

------

#### The inline keyword, historically

Historically, compilers either didn’t have the capability to determine whether inline expansion would be beneficial, or were not very good at it. For this reason, C++ provides the keyword `inline`, which was intended to be used as a hint to the compiler that a function would benefit from being expanded inline:

```cpp
#include <iostream>

inline int min(int x, int y) // hint to the compiler that it should do inline expansion of this function
{
    return (x < y) ? x : y;
}

int main()
{
    std::cout << min(5, 6) << '\n';
    std::cout << min(3, 2) << '\n';
    return 0;
}
```

This is where the term “inline function” comes from (because such functions had the `inline` specifier as part of the declaration syntax of the function).

However, in modern C++, the `inline` keyword is no longer used to request that a function be expanded inline. There are quite a few reasons for this:

- Using `inline` to request inline expansion is a form of premature optimization, and misuse could actually harm performance.
- The `inline` keyword is just a hint -- the compiler is completely free to ignore a request to inline a function. This is likely to be the result if you try to inline a lengthy function! The compiler is also free to perform inline expansion of functions that do not use the `inline` keyword as part of its normal set of optimizations.
- The `inline` keyword is defined at the wrong level of granularity. We use the `inline` keyword on a function declaration, but inline expansion is actually determined per function call. It may be beneficial to expand some function calls and detrimental to expand others, and there is no syntax to affect this.

Modern optimizing compilers are typically very good at determining which functions should be made inline -- better than humans in most cases. As a result, the compiler will likely ignore or devalue any request you make to `inline` a function anyway.

**Best practice**

Do not use the `inline` keyword to request inline expansion for your functions.

------

#### The inline keyword, modernly

In previous chapters, we mentioned that you should not implement functions (with external linkage) in header files, because when those headers are included into multiple .cpp files, the function definition will be copied into multiple .cpp files. These files will then be compiled, and the linker will throw an error because it will note that you’ve defined the same function more than once, which is a violation of the one-definition rule.

In lesson [6.9 -- Sharing global constants across multiple files (using inline variables)](https://www.learncpp.com/cpp-tutorial/sharing-global-constants-across-multiple-files-using-inline-variables/), we noted that in modern C++, the `inline` concept has evolved to have a new meaning: multiple definitions are allowed in the program. This is true for functions as well as variables. Thus, if we mark a function as inline, then that function is allowed to have multiple definitions (in different files), as long as those definitions are identical.

In order to do inline expansion, the compiler needs to be able to see the full definition of an inline function wherever the function is called. Therefore, inline functions are typically defined in header files, where they can be #included into any code file that needs to see the full definition of the function.

**Key insight**

The compiler needs to be able to see the full definition of an inline function wherever it is called.

For the most part, you should not mark your functions as inline, but we’ll see examples in the future where this is useful.

**Best practice**

Avoid the use of the `inline` keyword for functions unless you have a specific, compelling reason to do so.

------

### 6.14 — Constexpr and consteval functions

In lesson [4.13 -- Const variables and symbolic constants](https://www.learncpp.com/cpp-tutorial/const-variables-and-symbolic-constants/), we introduced the `constexpr` keyword, which we used to create compile-time (symbolic) constants. We also introduced constant expressions, which are expressions that can be evaluated at compile-time rather than runtime.

Consider the following program, which uses two constexpr variables:

```cpp
#include <iostream>

int main()
{
    constexpr int x{ 5 };
    constexpr int y{ 6 };

    std::cout << (x > y ? x : y) << " is greater!\n";

    return 0;
}
```

This produces the result:

```
6 is greater!
```

Because `x` and `y` are constexpr, the compiler can evaluate the constant expression `(x > y ? x : y)` at compile-time, reducing it to just `6`. Because this expression no longer needs to be evaluated at runtime, our program will run faster.

However, having a non-trivial expression in the middle of our print statement isn’t ideal -- it would be better if the expression were a named function. Here’s the same example using a function:

```cpp
#include <iostream>

int greater(int x, int y)
{
    return (x > y ? x : y); // here's our expression
}

int main()
{
    constexpr int x{ 5 };
    constexpr int y{ 6 };

    std::cout << greater(x, y) << " is greater!\n"; // will be evaluated at runtime

    return 0;
}
```

This program produces the same output as the prior one. But there’s a downside to putting our expression in a function: the call to `greater(x, y)` will execute at runtime. By using a function (which is good for modularity and documentation) we’ve lost our ability for that code to be evaluated at compile-time (which is bad for performance).

So how might we address this?

------

#### Constexpr functions can be evaluated at compile-time

A **constexpr function** is a function whose return value may be computed at compile-time. To make a function a constexpr function, we simply use the `constexpr` keyword in front of the return type. Here’s a similar program to the one above, using a constexpr function:

```cpp
#include <iostream>

constexpr int greater(int x, int y) // now a constexpr function
{
    return (x > y ? x : y);
}

int main()
{
    constexpr int x{ 5 };
    constexpr int y{ 6 };

    // We'll explain why we use variable g here later in the lesson
    constexpr int g { greater(x, y) }; // will be evaluated at compile-time

    std::cout << g << " is greater!\n";

    return 0;
}
```

This produces the same output as the prior example, but the function call `greater(x, y)` will be evaluated at compile-time instead of runtime!

When a function call is evaluated at compile-time, the compiler will calculate the return value of the function call, and then replace the function call with the return value.

So in our example, the call to `greater(x, y)` will be replaced by the result of the function call, which is the integer value `6`. In other words, the compiler will compile this:

```cpp
#include <iostream>

int main()
{
    constexpr int x{ 5 };
    constexpr int y{ 6 };

    constexpr int g { 6 }; // greater(x, y) evaluated and replaced with return value 6

    std::cout << g << " is greater!\n";

    return 0;
}
```

To be eligible for compile-time evaluation, a function must have a constexpr return type and not call any non-constexpr functions. Additionally, a call to the function must have constexpr arguments (e.g. constexpr variables or literals).

**Author’s note**

We’ll use the term “eligible for compile-time evaluation” later in the article, so remember this definition.

**For advanced readers**

There are some other lesser encountered criteria as well. These can be found [here](https://en.cppreference.com/w/cpp/language/constexpr).

Our `greater()` function definition and function call in the above example meets these requirements, so it is eligible for compile-time evaluation.

**Best practice**

Use a `constexpr` return type for functions that need to return a compile-time constant.

------

#### Constexpr functions are implicitly inline

Because constexpr functions may be evaluated at compile-time, the compiler must be able to see the full definition of the constexpr function at all points where the function is called. A forward declaration will not suffice, even if the actual function definition appears later in the same compilation unit.

This means that a constexpr function called in multiple files needs to have its definition included into each such file -- which would normally be a violation of the one-definition rule. To avoid such problems, constexpr functions are implicitly inline, which makes them exempt from the one-definition rule.

As a result, constexpr functions are often defined in header files, so they can be #included into any .cpp file that requires the full definition.

**Rule**

The compiler must be able to see the full definition of a constexpr function, not just a forward declaration.

**Best practice**

Constexpr functions used in a single source file (.cpp) can be defined in the source file above where they are used.

Constexpr functions used in multiple source files should be defined in a header file so they can be included into each source file.

------

#### Constexpr functions can also be evaluated at runtime

Functions with a constexpr return value can also be evaluated at runtime, in which case they will return a non-constexpr result. For example:

```cpp
#include <iostream>

constexpr int greater(int x, int y)
{
    return (x > y ? x : y);
}

int main()
{
    int x{ 5 }; // not constexpr
    int y{ 6 }; // not constexpr

    std::cout << greater(x, y) << " is greater!\n"; // will be evaluated at runtime

    return 0;
}
```

In this example, because arguments `x` and `y` are not constexpr, the function cannot be resolved at compile-time. However, the function will still be resolved at runtime, returning the expected value as a non-constexpr `int`.

**Key insight**

Allowing functions with a constexpr return type to be evaluated at either compile-time or runtime was allowed so that a single function can serve both cases.

Otherwise, you’d need to have separate functions (a function with a constexpr return type, and a function with a non-constexpr return type). This would not only require duplicate code, the two functions would also need to have different names!

A constexpr function is not allowed to call a non-constexpr function. If this were allowed, the constexpr function wouldn’t be able to evaluate at compile-time, which defeats the point of constexpr. Trying to do so will cause the compiler to produce a compilation error.

------

#### So when is a constexpr function evaluated at compile-time?

You might think that a constexpr function would evaluate at compile-time whenever possible, but unfortunately this is not the case.

According to the C++ standard, a constexpr function that is eligible for compile-time evaluation *must* be evaluated at compile-time if the return value is used where a constant expression is required. Otherwise, the compiler is free to evaluate the function at either compile-time or runtime.

Let’s examine a few cases to explore this further:

```cpp
#include <iostream>

constexpr int greater(int x, int y)
{
    return (x > y ? x : y);
}

int main()
{
    constexpr int g { greater(5, 6) };            // case 1: evaluated at compile-time
    std::cout << g << " is greater!\n";

    int x{ 5 }; // not constexpr
    std::cout << greater(x, 6) << " is greater!\n"; // case 2: evaluated at runtime

    std::cout << greater(5, 6) << " is greater!\n"; // case 3: may be evaluated at either runtime or compile-time

    return 0;
}
```

In case 1, we’re calling `greater()` with constexpr arguments, so it is eligible to be evaluated at compile-time. The initializer of constexpr variable `g` must be a constant expression, so the return value is used in a context that requires a constant expression. Thus, `greater()` must be evaluated at compile-time.

In case 2, we’re calling `greater()` with one parameter that is non-constexpr. Thus `greater()` cannot be evaluated at compile-time, and must evaluate at runtime.

Case 3 is the interesting case. The `greater()` function is again being called with constexpr arguments, so it is eligible for compile-time evaluation. However, the return value is not being used in a context that requires a constant expression (operator<< always executes at runtime), so the compiler is free to choose whether this call to `greater()` will be evaluated at compile-time or runtime!

Note that your compiler’s optimization level setting may have an impact on whether it decides to evaluate a function at compile-time or runtime. This also means that your compiler may make different choices for debug vs. release builds (as debug builds typically have optimizations turned off).

**Key insight**

A constexpr function that is eligible to be evaluated at compile-time will only be evaluated at compile-time if the return value is used where a constant expression is required. Otherwise, compile-time evaluation is not guaranteed.

Thus, a constexpr function is better thought of as “can be used in a constant expression”, not “will be evaluated at compile-time”.

------

#### Determining if a constexpr function call is evaluating at compile-time or runtime

Prior to C++20, there are no standard language tools available to do this.

In C++20, `std::is_constant_evaluated()` (defined in the <type_traits> header) returns a `bool` indicating whether the current function call is executing in a constant context. This can be combined with a conditional statement to allow a function to behave differently when evaluated at compile-time vs runtime.

```cpp
#include <type_traits> // for std::is_constant_evaluated
constexpr int someFunction()
{
    if (std::is_constant_evaluated()) // if compile-time evaluation
        // do something
    else // runtime evaluation
        // do something else
}
```

Used cleverly, you can have your function produce some observable difference (such as returning a special value) when evaluated at compile-time, and then infer how it evaluated from that result.

------

#### Forcing a constexpr function to be evaluated at compile-time

There is no way to tell the compiler that a constexpr function should prefer to evaluate at compile-time whenever it can (even in cases where the return value is used in a non-constant expression).

However, we can force a constexpr function that is eligible to be evaluated at compile-time to actually evaluate at compile-time by ensuring the return value is used where a constant expression is required. This needs to be done on a per-call basis.

The most common way to do this is to use the return value to initialize a constexpr variable (this is why we’ve been using variable ‘g’ in prior examples). Unfortunately, this requires introducing a new variable into our program just to ensure compile-time evaluation, which is ugly and reduces code readability.

**For advanced readers**

There are several hacky ways that people have tried to work around the problem of having to introduce a new constexpr variable each time we want to force compile-time evaluation. See [here](https://quuxplusone.github.io/blog/2018/08/07/force-constexpr/) and [here](https://artificial-mind.net/blog/2020/11/14/cpp17-consteval).

However, in C++20, there is a better workaround to this issue, which we’ll present in a moment.

------

#### Consteval C++20

C++20 introduces the keyword **consteval**, which is used to indicate that a function *must* evaluate at compile-time, otherwise a compile error will result. Such functions are called **immediate functions**.

```cpp
#include <iostream>

consteval int greater(int x, int y) // function is now consteval
{
    return (x > y ? x : y);
}

int main()
{
    constexpr int g { greater(5, 6) };            // ok: will evaluate at compile-time
    std::cout << greater(5, 6) << " is greater!\n"; // ok: will evaluate at compile-time

    int x{ 5 }; // not constexpr
    std::cout << greater(x, 6) << " is greater!\n"; // error: consteval functions must evaluate at compile-time

    return 0;
}
```

In the above example, the first two calls to `greater()` will evaluate at compile-time. The call to `greater(x, 6)` cannot be evaluated at compile-time, so a compile error will result.

Just like constexpr functions, consteval functions are implicitly inline.

**Best practice**

Use `consteval` if you have a function that must run at compile-time for some reason (e.g. performance).

------

#### Using consteval to make constexpr execute at compile-time C++20

The downside of consteval functions is that such functions can’t evaluate at runtime, making them less flexible than constexpr functions, which can do either. Therefore, it would still be useful to have a convenient way to force constexpr functions to evaluate at compile-time (even when the return value is being used where a constant expression is not required), so that we could have compile-time evaluation when possible, and runtime evaluation when we can’t.

Consteval functions provides a way to make this happen, using a neat helper function:

```cpp
#include <iostream>

// Uses abbreviated function template (C++20) and `auto` return type to make this function work with any type of value
// See 'related content' box below for more info (you don't need to know how these work to use this function)
consteval auto compileTime(auto value)
{
    return value;
}

constexpr int greater(int x, int y) // function is constexpr
{
    return (x > y ? x : y);
}

int main()
{
    std::cout << greater(5, 6) << '\n';              // may or may not execute at compile-time
    std::cout << compileTime(greater(5, 6)) << '\n'; // will execute at compile-time

    int x { 5 };
    std::cout << greater(x, 6) << '\n';              // we can still call the constexpr version at runtime if we wish

    return 0;
}
```

This works because consteval functions require constant expressions as arguments -- therefore, if we use the return value of a constexpr function as an argument to a consteval function, the constexpr function must be evaluated at compile-time! The consteval function just returns this argument as its own return value, so the caller can still use it.

Note that the consteval function returns by value. While this might be inefficient to do at runtime (if the value was some type that is expensive to copy, e.g. std::string), in a compile-time context, it doesn’t matter because the entire call to the consteval function will simply be replaced with the calculated return value.

**Related content**

We cover `auto` return types in lesson [8.9 -- Type deduction for functions](https://www.learncpp.com/cpp-tutorial/type-deduction-for-functions/).
We cover abbreviated function templates (`auto` parameters) in lesson [8.16 -- Function templates with multiple template types](https://www.learncpp.com/cpp-tutorial/function-templates-with-multiple-template-types/).

------

### 6.15 — Unnamed and inline namespaces

C++ supports two variants of namespaces that are worth at least knowing about. We won’t build on these, so consider this lesson optional for now.

------

#### Unnamed (anonymous) namespaces

An **unnamed namespace** (also called an **anonymous namespace**) is a namespace that is defined without a name, like so:

```cpp
#include <iostream>

namespace // unnamed namespace
{
    void doSomething() // can only be accessed in this file
    {
        std::cout << "v1\n";
    }
}

int main()
{
    doSomething(); // we can call doSomething() without a namespace prefix

    return 0;
}
```

This prints:

```
v1
```

All content declared in an `unnamed namespace` is treated as if it is part of the parent namespace. So even though function `doSomething` is defined in the `unnamed namespace`, the function itself is accessible from the parent namespace (which in this case is the `global namespace`), which is why we can call `doSomething` from `main` without any qualifiers.

This might make `unnamed namespaces` seem useless. But the other effect of `unnamed namespaces` is that all identifiers inside an `unnamed namespace` are treated as if they had `internal linkage`, which means that the content of an `unnamed namespace` can’t be seen outside of the file in which the `unnamed namespace` is defined.

For functions, this is effectively the same as defining all functions in the `unnamed namespace` as `static functions`. The following program is effectively identical to the one above:

```cpp
#include <iostream>

static void doSomething() // can only be accessed in this file
{
    std::cout << "v1\n";
}

int main()
{
    doSomething(); // we can call doSomething() without a namespace prefix

    return 0;
}
```

`Unnamed namespaces` are typically used when you have a lot of content that you want to ensure stays local to a given file, as it’s easier to cluster such content in an `unnamed namespace` than individually mark all declarations as `static`. `Unnamed namespaces` will also keep `user-defined types` (something we’ll discuss in a later lesson) local to the file, something for which there is no alternative equivalent mechanism to do.

------

#### Inline namespaces

Now consider the following program:

```cpp
#include <iostream>

void doSomething()
{
    std::cout << "v1\n";
}

int main()
{
    doSomething();

    return 0;
}
```

This prints:

```
v1
```

Pretty straightforward, right?

But let’s say you’re not happy with `doSomething`, and you want to improve it in some way that changes how it behaves. But if you do this, you risk breaking existing programs using the older version. How do you handle this?

One way would be to create a new version of the function with a different name. But over the course of many changes, you could end up with a whole set of almost-identically named functions (`doSomething`, `doSomething_v2`, `doSomething_v3`, etc…).

An alternative is to use an inline namespace. An **inline namespace** is a namespace that is typically used to version content. Much like an `unnamed namespace`, anything declared inside an `inline namespace` is considered part of the parent namespace. However, `inline namespaces` don’t give everything `internal linkage`.

To define an inline namespace, we use the `inline` keyword:

```cpp
#include <iostream>

inline namespace v1 // declare an inline namespace named v1
{
    void doSomething()
    {
        std::cout << "v1\n";
    }
}

namespace v2 // declare a normal namespace named v2
{
    void doSomething()
    {
        std::cout << "v2\n";
    }
}

int main()
{
    v1::doSomething(); // calls the v1 version of doSomething()
    v2::doSomething(); // calls the v2 version of doSomething()

    doSomething(); // calls the inline version of doSomething() (which is v1)

    return 0;
}
```

This prints:

```
v1
v2
v1
```

In the above example, callers to `doSomething` will get the v1 (the inline version) of `doSomething`. Callers who want to use the newer version can explicitly call `v2::dosomething()`. This preserves the function of existing programs while allowing newer programs to take advantage of newer/better variations.

Alternatively, if you want to push the newer version:

```cpp
#include <iostream>

namespace v1 // declare a normal namespace named v1
{
    void doSomething()
    {
        std::cout << "v1\n";
    }
}

inline namespace v2 // declare an inline namespace named v2
{
    void doSomething()
    {
        std::cout << "v2\n";
    }
}

int main()
{
    v1::doSomething(); // calls the v1 version of doSomething()
    v2::doSomething(); // calls the v2 version of doSomething()

    doSomething(); // calls the inline version of doSomething() (which is v2)

    return 0;
}
```

This prints:

```
v1
v2
v2
```

In this example, all callers to `doSomething` will get the v2 version by default (the newer and better version). Users who still want the older version of `doSomething` can explicitly call `v1::doSomething()` to access the old behavior. This means existing programs who want the v1 version will need to globally replace `doSomething` with `v1::doSomething`, but this typically won’t be problematic if the functions are well named.

------

### 6.x — Chapter 6 summary and quiz

#### Quick review

We covered a lot of material in this chapter. Good job, you’re doing great!

A **compound statement** or **block** is a group of zero or more statements that is treated by the compiler as if it were a single statement. Blocks begin with a `{` symbol, end with a `}` symbol, with the statements to be executed placed in between. Blocks can be used anywhere a single statement is allowed. No semicolon is needed at the end of a block. Blocks are often used in conjunction with `if statements` to execute multiple statements.

**User-defined namespaces** are namespaces that are defined by you for your own declarations. Namespaces provided by C++ (such as the `global namespace`) or by libraries (such as `namespace std`) are not considered user-defined namespaces.

You can access a declaration in a namespace via the **scope resolution operator (::)**. The scope resolution operator tells the compiler that identifier specified by the right-hand operand should be looked for in the scope of the left-hand operand. If no left-hand operand is provided, the global namespace is assumed.

Local variables are variables defined within a function (including function parameters). Local variables have **block scope**, meaning they are in-scope from their point of definition to the end of the block they are defined within. Local variables have **automatic storage duration**, meaning they are created at the point of definition and destroyed at the end of the block they are defined in.

A name declared in a nested block can **shadow** or **name hide** an identically named variable in an outer block. This should be avoided.

Global variables are variables defined outside of a function. Global variables have **file scope**, which means they are visible from the point of declaration until the end of the file in which they are declared. Global variables have **static duration**, which means they are created when the program starts, and destroyed when it ends. Avoid dynamic initialization of static variables whenever possible.

An identifier’s **linkage** determines whether other declarations of that name refer to the same object or not. Local variables have no linkage. Identifiers with **internal linkage** can be seen and used within a single file, but it is not accessible from other files. Identifiers with **external linkage** can be seen and used both from the file in which it is defined, and from other code files (via a forward declaration).

Avoid non-const global variables whenever possible. Const globals are generally seen as acceptable. Use **inline variables** for global constants if your compiler is C++17 capable.

Local variables can be given static duration via the **static** keyword.

A **qualified name** is a name that includes an associated scope (e.g. `std::string`). An **unqualified name** is a name that does not include a scoping qualifier (e.g. `string`).

**Using statements** (including **using declarations** and **using directives**) can be used to avoid having to qualify identifiers with an explicit namespace. A **using declaration** allows us to use an unqualified name (with no scope) as an alias for a qualified name. A **using directive** imports all of the identifiers from a namespace into the scope of the using directive. Both of these should generally be avoided.

**Inline functions** were originally designed as a way to request that the compiler replace your function call with inline expansion of the function code. You should not need to use the inline keyword for this purpose because the compiler will generally determine this for you. In modern C++, the `inline` keyword is used to exempt a function from the one-definition rule, allowing its definition to be imported into multiple code files. Inline functions are typically defined in header files so they can be #included into any code files that needs them.

A **constexpr function** is a function whose return value may be computed at compile-time. To make a function a constexpr function, we simply use the `constexpr` keyword in front of the return type. A constexpr function that is eligible for compile-time evaluation must be evaluated at compile-time if the return value is used in a context that requires a constexpr value. Otherwise, the compiler is free to evaluate the function at either compile-time or runtime.

C++20 introduces the keyword `consteval`, which is used to indicate that a function must evaluate at compile-time, otherwise a compile error will result. Such functions are called **immediate functions**.

Finally, C++ supports **unnamed namespaces**, which implicitly treat all contents of the namespace as if it had internal linkage. C++ also supports **inline namespaces**, which provide some primitive versioning capabilities for namespaces.

------

#### Quiz time

**Question #1**

Fix the following program:

```cpp
#include <iostream>

int main()
{
	std::cout << "Enter a positive number: ";
	int num{};
	std::cin >> num;


	if (num < 0)
		std::cout << "Negative number entered.  Making positive.\n";
		num = -num;

	std::cout << "You entered: " << num;

	return 0;
}
```

```cpp
#include <iostream>

int main()
{
	std::cout << "Enter a positive number: ";
	int num{};
	std::cin >> num;


	if (num < 0)
	{ // block needed here so both statements execute if num is < 0
		std::cout << "Negative number entered.  Making positive.\n";
		num = -num;
	}

	std::cout << "You entered: " << num;

	return 0;
}
```

------

**Question #2**

Write a file named constants.h that makes the following program run. If your compiler is C++17 capable, use inline constexpr variables. Otherwise, use normal constexpr variables. `max_class_size` should be `35`.

main.cpp:

```cpp
#include <iostream>
#include "constants.h"

int main()
{
	std::cout << "How many students are in your class? ";
	int students{};
	std::cin >> students;


	if (students > constants::max_class_size)
		std::cout << "There are too many students in this class";
	else
		std::cout << "This class isn't too large";

	return 0;
}
```

constants.h:

```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace constants
{
	inline constexpr int max_class_size{ 35 }; // remove inline keyword if not C++17 capable
}
#endif
```

main.cpp:

```cpp
#include <iostream>
#include "constants.h"

int main()
{
	std::cout << "How many students are in your class? ";
	int students{};
	std::cin >> students;


	if (students > constants::max_class_size)
		std::cout << "There are too many students in this class";
	else
		std::cout << "This class isn't too large";

	return 0;
}
```

------

**Question #3**

Complete the following program by writing the passOrFail() function, which should return true for the first 3 calls, and false thereafter. Do this without modifying the main() function.

Hint: Use a static local variable to remember how many times passOrFail() has been called previously.

```cpp
#include <iostream>

int main()
{
	std::cout << "User #1: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #2: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #3: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #4: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #5: " << (passOrFail() ? "Pass\n" : "Fail\n");

	return 0;
}
```

The program should produce the following output:

```
User #1: Pass
User #2: Pass
User #3: Pass
User #4: Fail
User #5: Fail
```

```cpp
#include <iostream>

// note: It should be mentioned that the following function is poorly designed for two reasons:
// 1) There's no way to reset s_passes, so the function can't be reused in a program
// 2) The function inscrutably returns a different value after a certain number of calls
bool passOrFail()
{
	static int s_passes{ 3 };
	// Only decrement if necessary, to prevent eventual overflow
	if (s_passes >= 0)
	    --s_passes;
	return (s_passes >= 0);
}

int main()
{
	std::cout << "User #1: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #2: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #3: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #4: " << (passOrFail() ? "Pass\n" : "Fail\n");
	std::cout << "User #5: " << (passOrFail() ? "Pass\n" : "Fail\n");

	return 0;
}
```

------

