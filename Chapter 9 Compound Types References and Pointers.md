## Compound Types: References and Pointers

### 9.1 — Introduction to compound data types

In lesson 4.1 -- Introduction to fundamental data types, we introduced the fundamental data types, which are the basic data types that C++ provides as part of the core language.

We’ve made much use of these fundamental types in our programs so far, especially the int data type. And while these fundamental types are extremely useful for straightforward uses, they don’t cover our full range of needs as we begin to do more complicated things.

For example, imagine you were writing a math program to multiply two fractions. How would you represent a fraction in your program? You might use a pair of integers (one for the numerator, one for the denominator), like this:

```cpp
#include <iostream>

int main()
{
    // Our first fraction
    int num1 {};
    int den1 {};

    // Our second fraction
    int num2 {};
    int den2 {};

    // Used to eat the slash between the numerator and denominator
    char ignore {};

    std::cout << "Enter a fraction: ";
    std::cin >> num1 >> ignore >> den1;

    std::cout << "Enter a fraction: ";
    std::cin >> num2 >> ignore >> den2;

    std::cout << "The two fractions multiplied: "
        << num1 * num2 << '/' << den1 * den2 << '\n';

    return 0;
}
```

And a run of this program:

```
Enter a fraction: 1/2
Enter a fraction: 3/4
The two fractions multiplied: 3/8
```

While this program works, it introduces a couple of challenges for us to improve upon. First, each pair of integers is only loosely linked -- outside of comments and the context of how they are used in the code, there’s little to suggest that each numerator and denominator pair are related. Second, following the DRY (don’t repeat yourself) principle, we should create a function to handle the user inputting a fraction (along with some error handling). However, functions can only return a single value, so how would we return the numerator and denominator back to the caller?

Now imagine another case where you’re writing a program that needs to keep a list of employee IDs. How might you do so? You might try something like this:

```cpp
int main()
{
    int id1 { 42 };
    int id2 { 57 };
    int id3 { 162 };
    // and so on
}
```

But what if you had 100 employees? First, you’d need to type in 100 variable names. And what if you needed to print them all? Or pass them to a function? We’d be in for a lot of typing. This simply doesn’t scale.

Clearly fundamental types will only carry us so far.

------

#### Compound data types

Fortunately, C++ supports a second set of data types called `compound data types`. **Compound data types** (also sometimes called **composite data types**) are data types that can be constructed from fundamental data types (or other compound data types). Each compound data type has its own unique properties as well.

As we’ll show in this chapter and future chapters, we can use compound data types to elegantly solve all of the challenges we presented above.

C++ supports the following compound types:

- Functions
- Arrays
- Pointer types:
  - Pointer to object
  - Pointer to function
- Pointer to member types:
  - Pointer to data member
  - Pointer to member function
- Reference types:
  - L-value references
  - R-value references
- Enumerated types:
  - Unscoped enumerations
  - Scoped enumerations
- Class types:
  - Structs
  - Classes
  - Unions

You’ve already been using a one compound type regularly: functions. For example, consider this function:

```cpp
void doSomething(int x, double y)
{
}
```

The type of this function is `void(int, double)`. Note that this type is composed of fundamental types, making it a compound type. Of course, functions also have their own special behaviors as well (e.g. being callable).

Because there’s a lot of material to cover here, we’ll do it over three chapters. In this chapter, we’ll cover some of the more straightforward compound types, including `l-value references`, and `pointers`. Next chapter, we’ll cover `unscoped enumerations`, `scoped enumerations`, and basic `structs`. Then, in the chapter beyond that, we’ll introduce class types and dig into some of the more useful `array` types. This includes `std::string` (introduced in lesson [4.17 -- Introduction to std::string](https://www.learncpp.com/cpp-tutorial/introduction-to-stdstring/)), which is actually a class type!

Got your game face on? Let’s go!

------

### 9.2 — Value categories (lvalues and rvalues)

Before we talk about our first compound type (lvalue references), we’re going to take a little detour and talk about what an `lvalue` is.

In lesson [1.10 -- Introduction to expressions](https://www.learncpp.com/cpp-tutorial/introduction-to-expressions/), we defined an expression as “a combination of literals, variables, operators, and function calls that can be executed to produce a singular value”.

For example:

```cpp
#include <iostream>

int main()
{
    std::cout << 2 + 3 << '\n'; // The expression 2 + 3 produces the value 5

    return 0;
}
```

In the above program, the expression `2 + 3` is evaluated to produce the value 5, which is then printed to the console.

In lesson [5.4 -- Increment/decrement operators, and side effects](https://www.learncpp.com/cpp-tutorial/increment-decrement-operators-and-side-effects/), we also noted that expressions can produce side effects that outlive the expression:

```cpp
#include <iostream>

int main()
{
    int x { 5 };
    ++x; // This expression statement has the side-effect of incrementing x
    std::cout << x << '\n'; // prints 6

    return 0;
}
```

In the above program, the expression `++x` increments the value of `x`, and that value remains changed even after the expression has finished evaluating.

Besides producing values and side effects, expressions can do one more thing: they can evaluate to objects or functions. We’ll explore this point further in just a moment.

------

#### The properties of an expression

To help determine how expressions should evaluate and where they can be used, all expressions in C++ have two properties: a type and a value category.

------

#### The type of an expression

The type of an expression is equivalent to the type of the value, object, or function that results from the evaluated expression. For example:

```cpp
#include <iostream>

int main()
{
    auto v1 { 12 / 4 }; // int / int => int
    auto v2 { 12.0 / 4 }; // double / int => double

    return 0;
}
```

For `v1`, the compiler will determine (at compile time) that a division with two `int` operands will produce an `int` result, so `int` is the type of this expression. Via type inference, `int` will then be used as the type of `v1`.

For `v2`, the compiler will determine (at compile time) that a division with a `double` operand and an `int` operand will produce a `double` result. Remember that arithmetic operators must have operands of matching types, so in this case, the `int` operand gets converted to a `double`, and a floating point division is performed. So `double` is the type of this expression.

The compiler can use the type of an expression to determine whether an expression is valid in a given context. For example:

```cpp
#include <iostream>

void print(int x)
{
    std::cout << x << '\n';
}

int main()
{
    print("foo"); // error: print() was expecting an int argument, we tried to pass in a string literal

    return 0;
}
```

In the above program, the `print(int)` function is expecting an `int` parameter. However, the type of the expression we’re passing in (the string literal `"foo"`) does not match, and no conversion can be found. So a compile error results.

Note that the type of an expression must be determinable at compile time (otherwise type checking and type deduction wouldn’t work) -- however, the value of an expression may be determined at either compile time (if the expression is constexpr) or runtime (if the expression is not constexpr).

------

#### The value category of an expression

Now consider the following program:

```cpp
int main()
{
    int x{};

    x = 5; // valid: we can assign 5 to x
    5 = x; // error: can not assign value of x to literal value 5

    return 0;
}
```

One of these assignment statements is valid (assigning value `5` to variable `x`) and one is not (what would it mean to assign the value of `x` to the literal value `5`?). So how does the compiler know which expressions can legally appear on either side of an assignment statement?

The answer lies in the second property of expressions: the `value category`. The **value category** of an expression (or subexpression) indicates whether an expression resolves to a value, a function, or an object of some kind.

Prior to C++11, there were only two possible value categories: `lvalue` and `rvalue`.

In C++11, three additional value categories (`glvalue`, `prvalue`, and `xvalue`) were added to support a new feature called `move semantics`.

**Author’s note**

In this lesson, we’ll stick to the pre-C++11 view of value categories, as this makes for a gentler introduction to value categories (and is all that we need for the moment). We’ll cover move semantics (and the additional three value categories) in a future chapter.

------

#### Lvalue and rvalue expressions

An **lvalue** (pronounced “ell-value”, short for “left value” or “locator value”, and sometimes written as “l-value”) is an expression that evaluates to an identifiable object or function (or bit-field).

The term “identity” is used by the C++ standard, but is not well-defined. An entity (such as an object or function) that has an identity can be differentiated from other similar entities (typically by comparing the addresses of the entity).

Entities with identities can be accessed via an identifier, reference, or pointer, and typically have a lifetime longer than a single expression or statement.

```cpp
#include <iostream>

int main()
{
    int x { 5 };
    int y { x }; // x is an lvalue expression

    return 0;
}
```

In the above program, the expression `x` is an lvalue expression as it evaluates to variable `x` (which has an identifier).

Since the introduction of constants into the language, lvalues come in two subtypes: a **modifiable lvalue** is an lvalue whose value can be modified. A **non-modifiable lvalue** is an lvalue whose value can’t be modified (because the lvalue is const or constexpr).

```cpp
#include <iostream>

int main()
{
    int x{};
    const double d{};

    int y { x }; // x is a modifiable lvalue expression
    const double e { d }; // d is a non-modifiable lvalue expression

    return 0;
}
```

An **rvalue** (pronounced “arr-value”, short for “right value”, and sometimes written as `r-value`) is an expression that is not an l-value. Commonly seen rvalues include literals (except C-style string literals, which are lvalues) and the return value of functions and operators. Rvalues aren’t identifiable (meaning they have to be used immediately), and only exist within the scope of the expression in which they are used.

```cpp
#include <iostream>

int return5()
{
    return 5;
}

int main()
{
    int x{ 5 }; // 5 is an rvalue expression
    const double d{ 1.2 }; // 1.2 is an rvalue expression

    int y { x }; // x is a modifiable lvalue expression
    const double e { d }; // d is a non-modifiable lvalue expression
    int z { return5() }; // return5() is an rvalue expression (since the result is returned by value)

    int w { x + 1 }; // x + 1 is an rvalue expression
    int q { static_cast<int>(d) }; // the result of static casting d to an int is an rvalue expression

    return 0;
}
```

You may be wondering why `return5()`, `x + 1`, and `static_cast<int>(d)` are rvalues: the answer is because these expressions produce temporary values that are not identifiable objects.

Now we can answer the question about why `x = 5` is valid but `5 = x` is not: an assignment operation requires the left operand of the assignment to be a modifiable lvalue expression, and the right operand to be an rvalue expression. The latter assignment (`5 = x`) fails because the left operand expression `5` isn’t an lvalue.

```cpp
int main()
{
    int x{};

    // Assignment requires the left operand to be a modifiable lvalue expression and the right operand to be an rvalue expression
    x = 5; // valid: x is a modifiable lvalue expression and 5 is an rvalue expression
    5 = x; // error: 5 is an rvalue expression and x is a modifiable lvalue expression

    return 0;
}
```

**Related content**

A full list of lvalue and rvalue expressions can be found [here](https://en.cppreference.com/w/cpp/language/value_category). In C++11, rvalues are broken into two subtypes: prvalues and xvalues, so the rvalues we’re talking about here are the sum of both of those categories.

------

#### L-value to r-value conversion

We said above that the assignment operator expects the right operand to be an rvalue expression, so why does code like this work?

```cpp
int main()
{
    int x{ 1 };
    int y{ 2 };

    x = y; // y is a modifiable lvalue, not an rvalue, but this is legal

    return 0;
}
```

The answer is because lvalues will implicitly convert to rvalues, so an lvalue can be used wherever an rvalue is required.

Now consider this snippet:

```cpp
int main()
{
    int x { 2 };

    x = x + 1;

    return 0;
}
```

In this statement, the variable `x` is being used in two different contexts. On the left side of the assignment operator, `x` is an lvalue expression that evaluates to variable x. On the right side of the assignment operator, `x + 1` is an rvalue expression that evaluates to the value `3`.

Now that we’ve covered lvalues, we can get to our first compound type: the `lvalue reference`.

**Key insight**

A rule of thumb to identify lvalue and rvalue expressions:

- Lvalue expressions are those that evaluate to variables or other identifiable objects that persist beyond the end of the expression.
- Rvalue expressions are those that evaluate to literals or values returned by functions/operators that are discarded at the end of the expression.

------

### 9.3 — Lvalue references

In C++, a **reference** is an alias for an existing object. Once a reference has been defined, any operation on the reference is applied to the object being referenced.

**Key insight**

A reference is essentially identical to the object being referenced.

This means we can use a reference to read or modify the object being referenced. Although references might seem silly, useless, or redundant at first, references are used everywhere in C++ (we’ll see examples of this in a few lessons).

You can also create references to functions, though this is done less often.

Modern C++ contains two types of references: `lvalue references`, and `rvalue references`. In this chapter, we’ll discuss lvalue references.

**Related content**

Because we’ll be talking about `lvalues` and `rvalues` in this lesson, please review [9.2 -- Value categories (lvalues and rvalues)](https://www.learncpp.com/cpp-tutorial/value-categories-lvalues-and-rvalues/) if you need a refresher on these terms before proceeding.

Rvalue references are covered in the chapter on `move semantics` ([chapter M](https://www.learncpp.com/#ChapterM)).

------

#### Lvalue reference types

An **lvalue reference** (commonly just called a `reference` since prior to C++11 there was only one type of reference) acts as an alias for an existing lvalue (such as a variable).

To declare an lvalue reference type, we use an ampersand (&) in the type declaration:

```cpp
int      // a normal int type
int&     // an lvalue reference to an int object
double&  // an lvalue reference to a double object
```

------

#### Lvalue reference variables

One of the things we can do with an lvalue reference type is create an lvalue reference variable. An **lvalue reference variable** is a variable that acts as a reference to an lvalue (usually another variable).

To create an lvalue reference variable, we simply define a variable with an lvalue reference type:

```cpp
#include <iostream>

int main()
{
    int x { 5 };    // x is a normal integer variable
    int& ref { x }; // ref is an lvalue reference variable that can now be used as an alias for variable x

    std::cout << x << '\n';  // print the value of x (5)
    std::cout << ref << '\n'; // print the value of x via ref (5)

    return 0;
}
```

In the above example, the type `int&` defines `ref` as an lvalue reference to an int, which we then initialize with lvalue expression `x`. Thereafter, `ref` and `x` can be used synonymously. This program thus prints:

```
5
5
```

From the compiler’s perspective, it doesn’t matter whether the ampersand is “attached” to the type name (`int& ref`) or the variable’s name (`int &ref`), and which you choose is a matter of style. Modern C++ programmers tend to prefer attaching the ampersand to the type, as it makes clearer that the reference is part of the type information, not the identifier.

**Best practice**

When defining a reference, place the ampersand next to the type (not the reference variable’s name).

**For advanced readers**

For those of you already familiar with pointers, the ampersand in this context does not mean “address of”, it means “lvalue reference to”.

------

#### Modifying values through an lvalue reference

In the above example, we showed that we can use a reference to read the value of the object being referenced. We can also use a reference to modify the value of the object being referenced:

```cpp
#include <iostream>

int main()
{
    int x { 5 }; // normal integer variable
    int& ref { x }; // ref is now an alias for variable x

    std::cout << x << ref << '\n'; // print 55

    x = 6; // x now has value 6

    std::cout << x << ref << '\n'; // prints 66

    ref = 7; // the object being referenced (x) now has value 7

    std::cout << x << ref << '\n'; // prints 77

    return 0;
}
```

This code prints:

```
55
66
77
```

In the above example, `ref` is an alias for `x`, so we are able to change the value of `x` through either `x` or `ref`.

------

#### Initialization of lvalue references

Much like constants, all references must be initialized.

```cpp
int main()
{
    int& invalidRef;   // error: references must be initialized

    int x { 5 };
    int& ref { x }; // okay: reference to int is bound to int variable

    return 0;
}
```

When a reference is initialized with an object (or function), we say it is **bound** to that object (or function). The process by which such a reference is bound is called **reference binding**. The object (or function) being referenced is sometimes called the **referent**.

Lvalue references must be bound to a *modifiable* lvalue.

```cpp
int main()
{
    int x { 5 };
    int& ref { x }; // valid: lvalue reference bound to a modifiable lvalue

    const int y { 5 };
    int& invalidRef { y };  // invalid: can't bind to a non-modifiable lvalue
    int& invalidRef2 { 0 }; // invalid: can't bind to an r-value

    return 0;
}
```

Lvalue references can’t be bound to non-modifiable lvalues or rvalues (otherwise you’d be able to change those values through the reference, which would be a violation of their const-ness). For this reason, lvalue references are occasionally called **lvalue references to non-const** (sometimes shortened to **non-const reference**).

In most cases, the type of the reference must match the type of the referent (there are some exceptions to this rule that we’ll discuss when we get into inheritance):

```cpp
int main()
{
    int x { 5 };
    int& ref { x }; // okay: reference to int is bound to int variable

    double y { 6.0 };
    int& invalidRef { y }; // invalid; reference to int cannot bind to double variable
    double& invalidRef2 { x }; // invalid: reference to double cannot bind to int variable

    return 0;
}
```

Lvalue references to `void` are disallowed (what would be the point?).

------

#### References can’t be reseated (changed to refer to another object)

Once initialized, a reference in C++ cannot be **reseated**, meaning it cannot be changed to reference another object.

New C++ programmers often try to reseat a reference by using assignment to provide the reference with another variable to reference. This will compile and run -- but not function as expected. Consider the following program:

```cpp
#include <iostream>

int main()
{
    int x { 5 };
    int y { 6 };

    int& ref { x }; // ref is now an alias for x

    ref = y; // assigns 6 (the value of y) to x (the object being referenced by ref)
    // The above line does NOT change ref into a reference to variable y!

    std::cout << x << '\n'; // user is expecting this to print 5

    return 0;
}
```

Perhaps surprisingly, this prints:

```
6
```

When a reference is evaluated in an expression, it resolves to the object it’s referencing. So `ref = y` doesn’t change `ref` to now reference `y`. Rather, because `ref` is an alias for `x`, the expression evaluates as if it was written `x = y` -- and since `y` evaluates to value `6`, `x` is assigned the value `6`.

------

#### Lvalue reference scope and duration

Reference variables follow the same scoping and duration rules that normal variables do:

```cpp
#include <iostream>

int main()
{
    int x { 5 }; // normal integer
    int& ref { x }; // reference to variable value

     return 0;
} // x and ref die here
```

------

#### References and referents have independent lifetimes

With one exception (that we’ll cover next lesson), the lifetime of a reference and the lifetime of its referent are independent. In other words, both of the following are true:

- A reference can be destroyed before the object it is referencing.
- The object being referenced can be destroyed before the reference.

When a reference is destroyed before the referent, the referent is not impacted. The following program demonstrates this:

```cpp
#include <iostream>

int main()
{
    int x { 5 };

    {
        int& ref { x };   // ref is a reference to x
        std::cout << ref << '\n'; // prints value of ref (5)
    } // ref is destroyed here -- x is unaware of this

    std::cout << x << '\n'; // prints value of x (5)

    return 0;
} // x destroyed here
```

The above prints:

```
5
5
```

When `ref` dies, variable `x` carries on as normal, blissfully unaware that a reference to it has been destroyed.

------

#### Dangling references

When an object being referenced is destroyed before a reference to it, the reference is left referencing an object that no longer exists. Such a reference is called a **dangling reference**. Accessing a dangling reference leads to undefined behavior.

Dangling references are fairly easy to avoid, but we’ll show a case where this can happen in practice in lesson [9.11 -- Return by reference and return by address](https://www.learncpp.com/cpp-tutorial/return-by-reference-and-return-by-address/).

------

#### References aren’t objects

Perhaps surprisingly, references are not objects in C++. A reference is not required to exist or occupy storage. If possible, the compiler will optimize references away by replacing all occurrences of a reference with the referent. However, this isn’t always possible, and in such cases, references may require storage.

This also means that the term “reference variable” is a bit of a misnomer, as variables are objects with a name, and references aren’t objects.

Because references aren’t objects, they can’t be used anywhere an object is required (e.g. you can’t have a reference to a reference, since an lvalue reference must reference an identifiable object). In cases where you need a reference that is an object or a reference that can be reseated, `std::reference_wrapper` (which we cover in lesson [16.3 -- Aggregation](https://www.learncpp.com/cpp-tutorial/aggregation/)) provides a solution.

**As an aside…**

Consider the following variables:

```cpp
int var{};
int& ref1{ var };  // an lvalue reference bound to var
int& ref2{ ref1 }; // an lvalue reference bound to var
```

Because `ref2` (a reference) is initialized with `ref1` (a reference), you might be tempted to conclude that `ref2` is a reference to a reference. It is not. Because `ref1` is a reference to `var`, when used in an expression (such as an initializer), `ref1` evaluates to `var`. So `ref2` is just a normal lvalue reference (as indicated by its type `int&`), bound to `var`.

A reference to a reference (to an `int`) would have syntax `int&&` -- but since C++ doesn’t support references to references, this syntax was repurposed in C++11 to indicate an rvalue reference (which we cover in lesson [M.2 -- R-value references](https://www.learncpp.com/cpp-tutorial/rvalue-references/)).

------

#### Quiz time

**Question #1**

Determine what values the following program prints by yourself (do not compile the program).

```cpp
#include <iostream>

int main()
{
    int x{ 1 };
    int& ref{ x };

    std::cout << x << ref << '\n';

    int y{ 2 };
    ref = y;
    y = 3;

    std::cout << x << ref << '\n';

    x = 4;

    std::cout << x << ref << '\n';

    return 0;
}
```

11
22
44

Because `ref` is bound to `x`, `x` and `ref` are synonymous, so they will always print the same value. The line `ref = y` assigns the value of `y` (2) to `ref` -- it does not change `ref` to reference `y`. The subsequent line `y = 3` only changes `y`.

------

### 9.4 — Lvalue references to const

In the previous lesson ([9.3 -- Lvalue references](https://www.learncpp.com/cpp-tutorial/lvalue-references/)), we discussed how an lvalue reference can only bind to a modifiable lvalue. This means the following is illegal:

```cpp
int main()
{
    const int x { 5 }; // x is a non-modifiable (const) lvalue
    int& ref { x }; // error: ref can not bind to non-modifiable lvalue

    return 0;
}
```

This is disallowed because it would allow us to modify a const variable (`x`) through the non-const reference (`ref`).

But what if we want to have a const variable we want to create a reference to? A normal lvalue reference (to a non-const value) won’t do.

------

#### Lvalue reference to const

By using the `const` keyword when declaring an lvalue reference, we tell an lvalue reference to treat the object it is referencing as const. Such a reference is called an **lvalue reference to a const value** (sometimes called a **reference to const** or a **const reference**).

Lvalue references to const can bind to non-modifiable lvalues:

```cpp
int main()
{
    const int x { 5 };    // x is a non-modifiable lvalue
    const int& ref { x }; // okay: ref is a an lvalue reference to a const value

    return 0;
}
```

Because lvalue references to const treat the object they are referencing as const, they can be used to access but not modify the value being referenced:

```cpp
#include <iostream>

int main()
{
    const int x { 5 };    // x is a non-modifiable lvalue
    const int& ref { x }; // okay: ref is a an lvalue reference to a const value

    std::cout << ref << '\n'; // okay: we can access the const object
    ref = 6;                  // error: we can not modify an object through a const reference

    return 0;
}
```

------

#### Initializing an lvalue reference to const with a modifiable lvalue

Lvalue references to const can also bind to modifiable lvalues. In such a case, the object being referenced is treated as const when accessed through the reference (even though the underlying object is non-const):

```cpp
#include <iostream>

int main()
{
    int x { 5 };          // x is a modifiable lvalue
    const int& ref { x }; // okay: we can bind a const reference to a modifiable lvalue

    std::cout << ref << '\n'; // okay: we can access the object through our const reference
    ref = 7;                  // error: we can not modify an object through a const reference

    x = 6;                // okay: x is a modifiable lvalue, we can still modify it through the original identifier

    return 0;
}
```

In the above program, we bind const reference `ref` to modifiable lvalue `x`. We can then use `ref` to access `x`, but because `ref` is const, we can not modify the value of `x` through `ref`. However, we still can modify the value of `x` directly (using the identifier `x`).

**Best practice**

Favor `lvalue references to const` over `lvalue references to non-const` unless you need to modify the object being referenced.

------

#### Initializing an lvalue reference to const with an rvalue

Perhaps surprisingly, lvalues references to const can also bind to rvalues:

```cpp
#include <iostream>

int main()
{
    const int& ref { 5 }; // okay: 5 is an rvalue

    std::cout << ref << '\n'; // prints 5

    return 0;
}
```

When this happens, a temporary object is created and initialized with the rvalue, and the reference to const is bound to that temporary object.

A **temporary object** (also sometimes called an **anonymous object**) is an object that is created for temporary use (and then destroyed) within a single expression. Temporary objects have no scope at all (this makes sense, since scope is a property of an identifier, and temporary objects have no identifier). This means a temporary object can only be used directly at the point where it is created, since there is no way to refer to it beyond that point.

------

#### Const references bound to temporary objects extend the lifetime of the temporary object

Temporary objects are normally destroyed at the end of the expression in which they are created.

However, consider what would happen in the above example if the temporary object created to hold rvalue `5` was destroyed at the end of the expression that initializes `ref`. Reference `ref` would be left dangling (referencing an object that had been destroyed), and we’d get undefined behavior when we tried to access `ref`.

To avoid dangling references in such cases, C++ has a special rule: When a const lvalue reference is bound to a temporary object, the lifetime of the temporary object is extended to match the lifetime of the reference.

```cpp
#include <iostream>

int main()
{
    const int& ref { 5 }; // The temporary object holding value 5 has its lifetime extended to match ref

    std::cout << ref << '\n'; // Therefore, we can safely use it here

    return 0;
} // Both ref and the temporary object die here
```

In the above example, when `ref` is initialized with rvalue `5`, a temporary object is created and `ref` is bound to that temporary object. The lifetime of the temporary object matches the lifetime of `ref`. Thus, we can safely print the value of `ref` in the next statement. Then both `ref` and the temporary object go out of scope and are destroyed at the end of the block.

**Key insight**

Lvalue references can only bind to modifiable lvalues.

Lvalue references to const can bind to modifiable lvalues, non-modifiable lvalues, and rvalues. This makes them a much more flexible type of reference.

So why does C++ allow a const reference to bind to an rvalue anyway? We’ll answer that question in the next lesson!

------

### 9.5 — Pass by lvalue reference

In the previous lessons, we introduced lvalue references ([9.3 -- Lvalue references](https://www.learncpp.com/cpp-tutorial/lvalue-references/)) and lvalue references to const ([9.4 -- Lvalue references to const](https://www.learncpp.com/cpp-tutorial/lvalue-references-to-const/)). In isolation, these may not have seemed very useful -- why create an alias to a variable when you can just use the variable itself?

In this lesson, we’ll finally provide some insight into what makes references useful. And then starting later in this chapter, you’ll see references used regularly.

First, some context. Back in lesson [2.4 -- Introduction to function parameters and arguments](https://www.learncpp.com/cpp-tutorial/introduction-to-function-parameters-and-arguments/) we discussed `pass by value`, where an argument passed to a function is copied into the function’s parameter:

```cpp
#include <iostream>

void printValue(int y)
{
    std::cout << y << '\n';
} // y is destroyed here

int main()
{
    int x { 2 };

    printValue(x); // x is passed by value (copied) into parameter y (inexpensive)

    return 0;
}
```

In the above program, when `printValue(x)` is called, the value of `x` (`2`) is *copied* into parameter `y`. Then, at the end of the function, object `y` is destroyed.

This means that when we called the function, we made a copy of our argument’s value, only to use it briefly and then destroy it! Fortunately, because fundamental types are cheap to copy, this isn’t a problem.

------

#### Some objects are expensive to copy

Most of the types provided by the standard library (such as `std::string`) are `class types`. Class types are usually expensive to copy. Whenever possible, we want to avoid making unnecessary copies of objects that are expensive to copy, especially when we will destroy those copies almost immediately.

Consider the following program illustrating this point:

```cpp
#include <iostream>
#include <string>

void printValue(std::string y)
{
    std::cout << y << '\n';
} // y is destroyed here

int main()
{
    std::string x { "Hello, world!" }; // x is a std::string

    printValue(x); // x is passed by value (copied) into parameter y (expensive)

    return 0;
}
```

This prints

```
Hello, world!
```

While this program behaves like we expect, it’s also inefficient. Identically to the prior example, when `printValue()` is called, argument `x` is copied into `printValue()` parameter `y`. However, in this example, the argument is a `std::string` instead of an `int`, and `std::string` is a class type that is expensive to copy. And this expensive copy is made every time `printValue()` is called!

We can do better.

------

#### Pass by reference

One way to avoid making an expensive copy of an argument when calling a function is to use `pass by reference` instead of `pass by value`. When using **pass by reference**, we declare a function parameter as a reference type (or const reference type) rather than as a normal type. When the function is called, each reference parameter is bound to the appropriate argument. Because the reference acts as an alias for the argument, no copy of the argument is made.

Here’s the same example as above, using pass by reference instead of pass by value:

```cpp
#include <iostream>
#include <string>

void printValue(std::string& y) // type changed to std::string&
{
    std::cout << y << '\n';
} // y is destroyed here

int main()
{
    std::string x { "Hello, world!" };

    printValue(x); // x is now passed by reference into reference parameter y (inexpensive)

    return 0;
}
```

This program is identical to the prior one, except the type of parameter `y` has been changed from `std::string` to `std::string&` (an lvalue reference). Now, when `printValue(x)` is called, lvalue reference parameter `y` is bound to argument `x`. Binding a reference is always inexpensive, and no copy of `x` needs to be made. Because a reference acts as an alias for the object being referenced, when `printValue()` uses reference `y`, it’s accessing the actual argument `x` (rather than a copy of `x`).

**Key insight**

Pass by reference allows us to pass arguments to a function without making copies of those arguments each time the function is called.

------

#### Pass by reference allows us to change the value of an argument

When an object is passed by value, the function parameter receives a copy of the argument. This means that any changes to the value of the parameter are made to the copy of the argument, not the argument itself:

```cpp
#include <iostream>

void addOne(int y) // y is a copy of x
{
    ++y; // this modifies the copy of x, not the actual object x
}

int main()
{
    int x { 5 };

    std::cout << "value = " << x << '\n';

    addOne(x);

    std::cout << "value = " << x << '\n'; // x has not been modified

    return 0;
}
```

In the above program, because value parameter `y` is a copy of `x`, when we increment `y`, this only affects `y`. This program outputs:

```
value = 5
value = 5
```

However, since a reference acts identically to the object being referenced, when using pass by reference, any changes made to the reference parameter *will* affect the argument:

```cpp
#include <iostream>

void addOne(int& y) // y is bound to the actual object x
{
    ++y; // this modifies the actual object x
}

int main()
{
    int x { 5 };

    std::cout << "value = " << x << '\n';

    addOne(x);

    std::cout << "value = " << x << '\n'; // x has been modified

    return 0;
}
```

This program outputs:

```
value = 5
value = 6
```

In the above example, `x` initially has value `5`. When `addOne(x)` is called, reference parameter `y` is bound to argument `x`. When the `addOne()` function increments reference `y`, it’s actually incrementing argument `x` from `5` to `6` (not a copy of `x`). This changed value persists even after `addOne()` has finished executing.

**Key insight**

Passing values by reference to non-const allows us to write functions that modify the value of arguments passed in.

The ability for functions to modify the value of arguments passed in can be useful. Imagine you’ve written a function that determines whether a monster has successfully attacked the player. If so, the monster should do some amount of damage to the player’s health. If you pass your player object by reference, the function can directly modify the health of the actual player object that was passed in. If you pass the player object by value, you could only modify the health of a copy of the player object, which isn’t as useful.

------

#### Pass by reference to non-const can only accept modifiable lvalue arguments

Because a reference to a non-const value can only bind to a modifiable lvalue (essentially a non-const variable), this means that pass by reference only works with arguments that are modifiable lvalues. In practical terms, this significantly limits the usefulness of pass by reference to non-const, as it means we can not pass const variables or literals. For example:

```cpp
#include <iostream>
#include <string>

void printValue(int& y) // y only accepts modifiable lvalues
{
    std::cout << y << '\n';
}

int main()
{
    int x { 5 };
    printValue(x); // ok: x is a modifiable lvalue

    const int z { 5 };
    printValue(z); // error: z is a non-modifiable lvalue

    printValue(5); // error: 5 is an rvalue

    return 0;
}
```

Fortunately, there’s an easy way around this.

------

#### Pass by const reference

Unlike a reference to non-const (which can only bind to modifiable lvalues), a reference to const can bind to modifiable lvalues, non-modifiable lvalues, and rvalues. Therefore, if we make our reference parameter const, then it will be able to bind to any type of argument:

```cpp
#include <iostream>
#include <string>

void printValue(const int& y) // y is now a const reference
{
    std::cout << y << '\n';
}

int main()
{
    int x { 5 };
    printValue(x); // ok: x is a modifiable lvalue

    const int z { 5 };
    printValue(z); // ok: z is a non-modifiable lvalue

    printValue(5); // ok: 5 is a literal rvalue

    return 0;
}
```

Passing by const reference offers the same primary benefit as pass by reference (avoiding making a copy of the argument), while also guaranteeing that the function can *not* change the value being referenced.

For example, the following is disallowed, because `ref` is const:

```cpp
void addOne(const int& ref)
{
    ++ref; // not allowed: ref is const
}
```

In most cases, we don’t want our functions modifying the value of arguments.

**Best practice**

Favor passing by const reference over passing by non-const reference unless you have a specific reason to do otherwise (e.g. the function needs to change the value of an argument).

Now we can understand the motivation for allowing const lvalue references to bind to rvalues: without that capability, there would be no way to pass literals (or other rvalues) to functions that used pass by reference!

------

#### Mixing pass by value and pass by reference

A function with multiple parameters can determine whether each parameter is passed by value or passed by reference individually.

For example:

```cpp
#include <string>

void foo(int a, int& b, const std::string& c)
{
}

int main()
{
    int x { 5 };
    const std::string s { "Hello, world!" };

    foo(5, x, s);

    return 0;
}
```

In the above example, the first argument is passed by value, the second by reference, and the third by const reference.

------

#### When to pass by reference

Because class types can be expensive to copy (sometimes significantly so), class types are usually passed by const reference instead of by value to avoid making an expensive copy of the argument. Fundamental types are cheap to copy, so they are typically passed by value.

**Best practice**

Pass fundamental types by value, and class (or struct) types by const reference.

------

#### The cost of pass by value vs pass by reference (advanced)

Not all class types need to be passed by reference. And you may be wondering why we don’t just pass everything by reference. In this section (which is optional reading), we discuss the cost of pass by value vs pass by reference, and refine our best practice as to when we should use each.

There are two key points that will help us understand when we should pass by value vs pass by reference:

First, the cost of copying an object is generally proportional to two things:

- The size of the object. Objects that use more memory take more time to copy.
- Any additional setup costs. Some class types do additional setup when they are instantiated (e.g. such as opening a file or database, or allocating a certain amount of dynamic memory to hold an object of a variable size). These setup costs must be paid each time an object is copied.

On the other hand, binding a reference to an object is always fast (about the same speed as copying a fundamental type).

Second, accessing an object through a reference is slightly more expensive than accessing an object through a normal variable identifier. With a variable identifier, the running program can just go to the memory address assigned to that variable and access the value directly. With a reference, there usually is an extra step: the program must first access the reference to determine which object is being referenced, and only then can it go to that memory address for that object and access the value. The compiler can also sometimes optimize code using objects passed by value more highly than code using objects passed by reference. This means code generated to access objects passed by reference is typically slower than the code generated for objects passed by value.

We can now answer the question of why we don’t pass everything by reference:

- For objects that are cheap to copy, the cost of copying is similar to the cost of binding, so we favor pass by value so the code generated will be faster.
- For objects that are expensive to copy, the cost of the copy dominates, so we favor pass by (const) reference to avoid making a copy.

**Best practice**

Prefer pass by value for objects that are cheap to copy, and pass by const reference for objects that are expensive to copy. If you’re not sure whether an object is cheap or expensive to copy, favor pass by const reference.

The last question then is, how do we define “cheap to copy”? There is no absolute answer here, as this varies by compiler, use case, and architecture. However, we can formulate a good rule of thumb: An object is cheap to copy if it uses 2 or fewer “words” of memory (where a “word” is approximated by the size of a memory address) and it has no setup costs.

The following program defines a function-like macro that can be used to determine if a type (or object) uses 2 or fewer memory addresses worth of memory:

```cpp
#include <iostream>

// Function-like macro that evaluates to true if the type (or object) uses 2 or fewer memory addresses worth of memory
#define isSmall(T) (sizeof(T) <= 2 * sizeof(void*))

struct S
{
    double a, b, c;
};

int main()
{
    std::cout << std::boolalpha; // print true or false rather than 1 or 0
    std::cout << isSmall(int) << '\n'; // true
    std::cout << isSmall(double) << '\n'; // true
    std::cout << isSmall(S) << '\n'; // false

    return 0;
}
```

**As an aside…**

We use a preprocessor function-like macro here so that we can provide either an object OR a type name as a parameter (normal functions disallow this).

However, it can be hard to know whether a class type object has setup costs or not. It’s best to assume that most standard library classes have setup costs, unless you know otherwise that they don’t.

**Tip**

An object of type T is cheap to copy if `sizeof(T) <= 2 * sizeof(void*)` and has no additional setup costs.

Common types that are cheap to copy include all of the fundamental types, enumerated types, and std::string_view.
Common types that are expensive to copy include std::array, std::string, std::vector, and std::ostream.

------

### 9.6 — Introduction to pointers

Pointers are one of C++’s historical boogeymen, and a place where many aspiring C++ learners have gotten stuck. However, as you’ll see shortly, pointers are nothing to be scared of.

In fact, pointers behave a lot like lvalue references. But before we explain that further, let’s do some setup.

**Related content**

If you’re rusty or not familiar with lvalue references, now would be a good time to review them. We cover lvalue references in lessons [9.3 -- Lvalue references](https://www.learncpp.com/cpp-tutorial/lvalue-references/), [9.4 -- Lvalue references to const](https://www.learncpp.com/cpp-tutorial/lvalue-references-to-const/), and [9.5 -- Pass by lvalue reference](https://www.learncpp.com/cpp-tutorial/pass-by-lvalue-reference/).

Consider a normal variable, like this one:

```cpp
char x {}; // chars use 1 byte of memory
```

Simplifying a bit, when the code generated for this definition is executed, a piece of memory from RAM will be assigned to this object. For the sake of example, let’s say that the variable `x` is assigned memory address `140`. Whenever we use variable `x` in an expression or statement, the program will go to memory address `140` to access the value stored there.

The nice thing about variables is that we don’t need to worry about what specific memory addresses are assigned, or how many bytes are required to store the object’s value. We just refer to the variable by its given identifier, and the compiler translates this name into the appropriately assigned memory address. The compiler takes care of all the addressing.

This is also true with references:

```cpp
int main()
{
    char x {}; // assume this is assigned memory address 140
    char& ref { x }; // ref is an lvalue reference to x (when used with a type, & means lvalue reference)

    return 0;
}
```

Because `ref` acts as an alias for `x`, whenever we use `ref`, the program will go to memory address `140` to access the value. Again the compiler takes care of the addressing, so that we don’t have to think about it.

------

#### The address-of operator (&)

Although the memory addresses used by variables aren’t exposed to us by default, we do have access to this information. The **address-of operator** (&) returns the memory address of its operand. This is pretty straightforward:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    std::cout << x << '\n';  // print the value of variable x
    std::cout << &x << '\n'; // print the memory address of variable x

    return 0;
}
```

On the author’s machine, the above program printed:

```
5
0027FEA0
```

In the above example, we use the address-of operator (&) to retrieve the address assigned to variable `x` and print that address to the console. Memory addresses are typically printed as hexadecimal values (we covered hex in lesson [4.15 -- Literals](https://www.learncpp.com/cpp-tutorial/literals/)), often without the 0x prefix.

For objects that use more than one byte of memory, address-of will return the memory address of the first byte used by the object.

**Tip**

The & symbol tends to cause confusion because it has different meanings depending on context:

- When following a type name, & denotes an lvalue reference: `int& ref`.
- When used in a unary context in an expression, & is the address-of operator: `std::cout << &x`.
- When used in a binary context in an expression, & is the Bitwise AND operator: `std::cout << x & y`.

------

#### The dereference operator (*)

Getting the address of a variable isn’t very useful by itself.

The most useful thing we can do with an address is access the value stored at that address. The **dereference operator** (*) (also occasionally called the **indirection operator**) returns the value at a given memory address as an lvalue:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    std::cout << x << '\n';  // print the value of variable x
    std::cout << &x << '\n'; // print the memory address of variable x

    std::cout << *(&x) << '\n'; // print the value at the memory address of variable x (parentheses not required, but make it easier to read)

    return 0;
}
```

On the author’s machine, the above program printed:

```
5
0027FEA0
5
```

This program is pretty simple. First we declare a variable `x` and print its value. Then we print the address of variable `x`. Finally, we use the dereference operator to get the value at the memory address of variable `x` (which is just the value of `x`), which we print to the console.

**Key insight**

Given a memory address, we can use the dereference operator (*) to get the value at that address (as an lvalue).

The address-of operator (&) and dereference operator (*) work as opposites: address-of gets the address of an object, and dereference gets the object at an address.

**Tip**

Although the dereference operator looks just like the multiplication operator, you can distinguish them because the dereference operator is unary, whereas the multiplication operator is binary.

Getting the memory address of a variable and then immediately dereferencing that address to get a value isn’t that useful either (after all, we can just use the variable to access the value).

But now that we have the address-of operator (&) and dereference operator (*) added to our toolkits, we’re ready to talk about pointers.

------

#### Pointers

A **pointer** is an object that holds a *memory address* (typically of another variable) as its value. This allows us to store the address of some other object to use later.

**As an aside…**

In modern C++, the pointers we are talking about here are sometimes called “raw pointers” or “dumb pointers”, to help differentiate them from “smart pointers” that were introduced into the language more recently. We cover smart pointers in [chapter M](https://www.learncpp.com/#ChapterM).

Much like reference types are declared using an ampersand (&) character, pointer types are declared using an asterisk (*):

```cpp
int;  // a normal int
int&; // an lvalue reference to an int value

int*; // a pointer to an int value (holds the address of an integer value)
```

To create a pointer variable, we simply define a variable with a pointer type:

```cpp
int main()
{
    int x { 5 };    // normal variable
    int& ref { x }; // a reference to an integer (bound to x)

    int* ptr;       // a pointer to an integer

    return 0;
}
```

Note that this asterisk is part of the declaration syntax for pointers, not a use of the dereference operator.

**Best practice**

When declaring a pointer type, place the asterisk next to the type name.

**Warning**

Although you generally should not declare multiple variables on a single line, if you do, the asterisk has to be included with each variable.

```cpp
int* ptr1, ptr2;   // incorrect: ptr1 is a pointer to an int, but ptr2 is just a plain int!
int* ptr3, * ptr4; // correct: ptr3 and ptr4 are both pointers to an int
```

Although this is sometimes used as an argument to not place the asterisk with the type name (instead placing it next to the variable name), it’s a better argument for avoiding defining multiple variables in the same statement.

------

#### Pointer initialization

Like normal variables, pointers are *not* initialized by default. A pointer that has not been initialized is sometimes called a **wild pointer**. Wild pointers contain a garbage address, and dereferencing a wild pointer will result in undefined behavior. Because of this, you should always initialize your pointers to a known value.

**Best practice**

Always initialize your pointers.

```cpp
int main()
{
    int x{ 5 };

    int* ptr;        // an uninitialized pointer (holds a garbage address)
    int* ptr2{};     // a null pointer (we'll discuss these in the next lesson)
    int* ptr3{ &x }; // a pointer initialized with the address of variable x

    return 0;
}
```

Since pointers hold addresses, when we initialize or assign a value to a pointer, that value has to be an address. Typically, pointers are used to hold the address of another variable (which we can get using the address-of operator (&)).

Once we have a pointer holding the address of another object, we can then use the dereference operator (*) to access the value at that address. For example:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    std::cout << x << '\n'; // print the value of variable x

    int* ptr{ &x }; // ptr holds the address of x
    std::cout << *ptr << '\n'; // use dereference operator to print the value at the address that ptr is holding (which is x's address)

    return 0;
}
```

This prints:

```
5
5
```

Conceptually, you can think of the above snippet like this:
![img](C:\Users\ECT200O\Desktop\learncpp\6-Pointer.webp)

This is where pointers get their name from -- `ptr` is holding the address of `x`, so we say that `ptr` is “pointing to” `x`.

**Author’s note**

A note on pointer nomenclature: “X pointer” (where X is some type) is a commonly used shorthand for “pointer to an X”. So when we say, “an integer pointer”, we really mean “a pointer to an integer”. This distinction will be valuable when we talk about const pointers.

Much like the type of a reference has to match the type of object being referred to, the type of the pointer has to match the type of the object being pointed to:

```cpp
int main()
{
    int i{ 5 };
    double d{ 7.0 };

    int* iPtr{ &i };     // ok: a pointer to an int can point to an int object
    int* iPtr2 { &d };   // not okay: a pointer to an int can't point to a double
    double* dPtr{ &d };  // ok: a pointer to a double can point to a double object
    double* dPtr2{ &i }; // not okay: a pointer to a double can't point to an int
}
```

With one exception that we’ll discuss next lesson, initializing a pointer with a literal value is disallowed:

```cpp
int* ptr{ 5 }; // not okay
int* ptr{ 0x0012FF7C }; // not okay, 0x0012FF7C is treated as an integer literal
```

------

#### Pointers and assignment

We can use assignment with pointers in two different ways:

1. To change what the pointer is pointing at (by assigning the pointer a new address)
2. To change the value being pointed at (by assigning the dereferenced pointer a new value)

First, let’s look at a case where a pointer is changed to point at a different object:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    int* ptr{ &x }; // ptr initialized to point at x

    std::cout << *ptr << '\n'; // print the value at the address being pointed to (x's address)

    int y{ 6 };
    ptr = &y; // // change ptr to point at y

    std::cout << *ptr << '\n'; // print the value at the address being pointed to (y's address)

    return 0;
}
```

The above prints:

```
5
6
```

In the above example, we define pointer `ptr`, initialize it with the address of `x`, and dereference the pointer to print the value being pointed to (`5`). We then use the assignment operator to change the address that `ptr` is holding to the address of `y`. We then dereference the pointer again to print the value being pointed to (which is now `6`).

Now let’s look at how we can also use a pointer to change the value being pointed at:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    int* ptr{ &x }; // initialize ptr with address of variable x

    std::cout << x << '\n';    // print x's value
    std::cout << *ptr << '\n'; // print the value at the address that ptr is holding (x's address)

    *ptr = 6; // The object at the address held by ptr (x) assigned value 6 (note that ptr is dereferenced here)

    std::cout << x << '\n';
    std::cout << *ptr << '\n'; // print the value at the address that ptr is holding (x's address)

    return 0;
}
```

This program prints:

```
5
5
6
6
```

In this example, we define pointer `ptr`, initialize it with the address of `x`, and then print the value of both `x` and `*ptr` (`5`). Because `*ptr` returns an lvalue, we can use this on the left hand side of an assignment statement, which we do to change the value being pointed at by `ptr` to `6`. We then print the value of both `x` and `*ptr` again to show that the value has been updated as expected.

**Key insight**

When we use a pointer without a dereference (`ptr`), we are accessing the address held by the pointer. Modifying this (`ptr = &y`) changes what the pointer is pointing at.

When we dereference a pointer (`*ptr`), we are accessing the object being pointed at. Modifying this (`*ptr = 6;`) changes the value of the object being pointed at.

------

#### Pointers behave much like lvalue references

Pointers and lvalue references behave similarly. Consider the following program:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    int& ref { x };  // get a reference to x
    int* ptr { &x }; // get a pointer to x

    std::cout << x;
    std::cout << ref;  // use the reference to print x's value (5)
    std::cout << *ptr << '\n'; // use the pointer to print x's value (5)

    ref = 6; // use the reference to change the value of x
    std::cout << x;
    std::cout << ref;  // use the reference to print x's value (6)
    std::cout << *ptr << '\n'; // use the pointer to print x's value (6)

    *ptr = 7; // use the pointer to change the value of x
    std::cout << x;
    std::cout << ref;  // use the reference to print x's value (7)
    std::cout << *ptr << '\n'; // use the pointer to print x's value (7)

    return 0;
}
```

This program prints:

```
555
666
777
```

In the above program, we create a normal variable `x` with value `5`, and then create an lvalue reference and a pointer to `x`. Next, we use the lvalue reference to change the value from `5` to `6`, and show that we can access that updated value via all three methods. Finally, we use the dereferenced pointer to change the value from `6` to `7`, and again show that we can access the updated value via all three methods.

Thus, both pointers and references provide a way to indirectly access another object. The primary difference is that with pointers, we need to explicitly get the address to point at, and we have to explicitly dereference the pointer to get the value. With references, the address-of and dereference happens implicitly.

There are some other differences between pointers and references worth mentioning:

- References must be initialized, pointers are not required to be initialized (but should be).
- References are not objects, pointers are.
- References can not be reseated (changed to reference something else), pointers can change what they are pointing at.
- References must always be bound to an object, pointers can point to nothing (we’ll see an example of this in the next lesson).
- References are “safe” (outside of dangling references), pointers are inherently dangerous (we’ll also discuss this in the next lesson).

------

#### The address-of operator returns a pointer

It’s worth noting that the address-of operator (&) doesn’t return the address of its operand as a literal. Instead, it returns a pointer containing the address of the operand, whose type is derived from the argument (e.g. taking the address of an `int` will return the address in an `int` pointer).

We can see this in the following example:

```cpp
#include <iostream>
#include <typeinfo>

int main()
{
	int x{ 4 };
	std::cout << typeid(&x).name() << '\n'; // print the type of &x

	return 0;
}
```

On Visual Studio, this printed:

```
int *
```

With gcc, this prints “pi” (pointer to int) instead. Because the result of typeid().name() is compiler-dependent, your compiler may print something different, but it will have the same meaning.

------

#### The size of pointers

The size of a pointer is dependent upon the architecture the executable is compiled for -- a 32-bit executable uses 32-bit memory addresses -- consequently, a pointer on a 32-bit machine is 32 bits (4 bytes). With a 64-bit executable, a pointer would be 64 bits (8 bytes). Note that this is true regardless of the size of the object being pointed to:

```cpp
#include <iostream>

int main() // assume a 32-bit application
{
    char* chPtr{};        // chars are 1 byte
    int* iPtr{};          // ints are usually 4 bytes
    long double* ldPtr{}; // long doubles are usually 8 or 12 bytes

    std::cout << sizeof(chPtr) << '\n'; // prints 4
    std::cout << sizeof(iPtr) << '\n';  // prints 4
    std::cout << sizeof(ldPtr) << '\n'; // prints 4

    return 0;
}
```

The size of the pointer is always the same. This is because a pointer is just a memory address, and the number of bits needed to access a memory address is constant.

------

#### Dangling pointers

Much like a dangling reference, a **dangling pointer** is a pointer that is holding the address of an object that is no longer valid (e.g. because it has been destroyed).

Dereferencing a dangling pointer (e.g. in order to print the value being pointed at) will lead to undefined behavior, as you are trying to access an object that is no longer valid.

Perhaps surprisingly, the standard says “Any other use of an invalid pointer *value* has implementation-defined behavior”. This means that you can assign an invalid pointer a new value, such as nullptr (because this doesn’t use the invalid pointer’s value). However, any other operations that use the invalid pointer’s value (such as copying or incrementing an invalid pointer) will yield implementation-defined behavior.

**Key insight**

Dereferencing an invalid pointer will lead to undefined behavior. Any other use of an invalid pointer value is implementation-defined.

Here’s an example of creating a dangling pointer:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    int* ptr{ &x };

    std::cout << *ptr << '\n'; // valid

    {
        int y{ 6 };
        ptr = &y;

        std::cout << *ptr << '\n'; // valid
    } // y goes out of scope, and ptr is now dangling

    std::cout << *ptr << '\n'; // undefined behavior from dereferencing a dangling pointer

    return 0;
}
```

The above program will probably print:

```
5
6
6
```

But it may not, as the object that `ptr` was pointing at went out of scope and was destroyed at the end of the inner block, leaving `ptr` dangling.

------

#### Conclusion

Pointers are variables that hold a memory address. They can be dereferenced using the dereference operator (*) to retrieve the value at the address they are holding. Dereferencing a wild or dangling (or null) pointer will result in undefined behavior and will probably crash your application.

Pointers are both more flexible than references and more dangerous. We’ll continue to explore this in the upcoming lessons.

------

#### Quiz time

**Question #1**

What values does this program print? Assume a short is 2 bytes, and a 32-bit machine.

```cpp
#include <iostream>

int main()
{
	short value{ 7 }; // &value = 0012FF60
	short otherValue{ 3 }; // &otherValue = 0012FF54

	short* ptr{ &value };

	std::cout << &value << '\n';
	std::cout << value << '\n';
	std::cout << ptr << '\n';
	std::cout << *ptr << '\n';
	std::cout << '\n';

	*ptr = 9;

	std::cout << &value << '\n';
	std::cout << value << '\n';
	std::cout << ptr << '\n';
	std::cout << *ptr << '\n';
	std::cout << '\n';

	ptr = &otherValue;

	std::cout << &otherValue << '\n';
	std::cout << otherValue << '\n';
	std::cout << ptr << '\n';
	std::cout << *ptr << '\n';
	std::cout << '\n';

	std::cout << sizeof(ptr) << '\n';
	std::cout << sizeof(*ptr) << '\n';

	return 0;
}
```

```
0012FF60
7
0012FF60
7

0012FF60
9
0012FF60
9

0012FF54
3
0012FF54
3

4
2
```

A short explanation about the 4 and the 2. A 32-bit machine means that pointers will be 32 bits in length, but sizeof() always prints the size in bytes. 32 bits is 4 bytes. Thus the `sizeof(ptr)` is 4. Because `ptr` is a pointer to a short, `*ptr` is a short. The size of a short in this example is 2 bytes. Thus the `sizeof(*ptr)` is 2.

------

**Question #2**

What’s wrong with this snippet of code?

```cpp
int value{ 45 };
int* ptr{ &value }; // declare a pointer and initialize with address of value
*ptr = &value; // assign address of value to ptr
```

The last line of the above snippet doesn’t compile.

Let’s examine this program in more detail.

The first line contains a standard variable definition, along with an initialization value. Nothing special here.

In the second line, we’re defining a new pointer named `ptr`, and initializing it with the address of `value`. Remember that in this context, the asterisk is part of the pointer declaration syntax, not a dereference. So this line is fine.

On line three, the asterisk represents a dereference, which is used to get the value that a pointer is pointing to. So this line says, “retrieve the value that `ptr` is pointing to (an integer), and overwrite it with the address of `value` (an address). That doesn’t make any sense -- you can’t assign an address to an integer!

The third line should be:

```cpp
ptr = &value;
```

This correctly assigns the address of variable value to the pointer.

------

### 9.7 — Null pointers

In the previous lesson ([9.6 -- Introduction to pointers](https://www.learncpp.com/cpp-tutorial/introduction-to-pointers/)), we covered the basics of pointers, which are objects that hold the address of another object. This address can be dereferenced using the dereference operator (*) to get the value at that address:

```cpp
#include <iostream>

int main()
{
    int x{ 5 };
    std::cout << x << '\n'; // print the value of variable x

    int* ptr{ &x }; // ptr holds the address of x
    std::cout << *ptr << '\n'; // use dereference operator to print the value at the address that ptr is holding (which is x's address)

    return 0;
}
```

The above example prints:

```
5
5
```

In the prior lesson, we also noted that pointers do not need to point to anything. In this lesson, we’ll explore such pointers (and the various implications of pointing to nothing) further.

------

#### Null pointers

Besides a memory address, there is one additional value that a pointer can hold: a null value. A **null value** (often shortened to **null**) is a special value that means something has no value. When a pointer is holding a null value, it means the pointer is not pointing at anything. Such a pointer is called a **null pointer**.

The easiest way to create a null pointer is to use value initialization:

```cpp
int main()
{
    int* ptr {}; // ptr is now a null pointer, and is not holding an address

    return 0;
}
```

**Best practice**

Value initialize your pointers (to be null pointers) if you are not initializing them with the address of a valid object.

Because we can use assignment to change what a pointer is pointing at, a pointer that is initially set to null can later be changed to point at a valid object:

```cpp
#include <iostream>

int main()
{
    int* ptr {}; // ptr is a null pointer, and is not holding an address

    int x { 5 };
    ptr = &x; // ptr now pointing at object x (no longer a null pointer)

    std::cout << *ptr << '\n'; // print value of x through dereferenced ptr

    return 0;
}
```

------

#### The nullptr keyword

Much like the keywords `true` and `false` represent Boolean literal values, the **nullptr** keyword represents a null pointer literal. We can use `nullptr` to explicitly initialize or assign a pointer a null value.

```cpp
int main()
{
    int* ptr { nullptr }; // can use nullptr to initialize a pointer to be a null pointer

    int value { 5 };
    int* ptr2 { &value }; // ptr2 is a valid pointer
    ptr2 = nullptr; // Can assign nullptr to make the pointer a null pointer

    someFunction(nullptr); // we can also pass nullptr to a function that has a pointer parameter

    return 0;
}
```

In the above example, we use assignment to set the value of `ptr2` to `nullptr`, making `ptr2` a null pointer.

**Best practice**

Use `nullptr` when you need a null pointer literal for initialization, assignment, or passing a null pointer to a function.

------

#### Dereferencing a null pointer results in undefined behavior

Much like dereferencing a dangling (or wild) pointer leads to undefined behavior, dereferencing a null pointer also leads to undefined behavior. In most cases, it will crash your application.

The following program illustrates this, and will probably crash or terminate your application abnormally when you run it (go ahead, try it, you won’t harm your machine):

```cpp
#include <iostream>

int main()
{
    int* ptr {}; // Create a null pointer
    std::cout << *ptr << '\n'; // Dereference the null pointer

    return 0;
}
```

Conceptually, this makes sense. Dereferencing a pointer means “go to the address the pointer is pointing at and access the value there”. A null pointer holds a null value, which semantically means the pointer is not pointing at anything. So what value would it access?

Accidentally dereferencing null and dangling pointers is one of the most common mistakes C++ programmers make, and is probably the most common reason that C++ programs crash in practice.

**Warning**

Whenever you are using pointers, you’ll need to be extra careful that your code isn’t dereferencing null or dangling pointers, as this will cause undefined behavior (probably an application crash).

------

#### Checking for null pointers

Much like we can use a conditional to test Boolean values for `true` or `false`, we can use a conditional to test whether a pointer has value `nullptr` or not:

```cpp
#include <iostream>

int main()
{
    int x { 5 };
    int* ptr { &x };

    // pointers convert to Boolean false if they are null, and Boolean true if they are non-null
    if (ptr == nullptr) // explicit test for equivalence
        std::cout << "ptr is null\n";
    else
        std::cout << "ptr is non-null\n";

    int* nullPtr {};
    std::cout << "nullPtr is " << (nullPtr==nullptr ? "null\n" : "non-null\n"); // explicit test for equivalence

    return 0;
}
```

The above program prints:

```
ptr is non-null
nullPtr is null
```

In lesson [4.9 -- Boolean values](https://www.learncpp.com/cpp-tutorial/boolean-values/), we noted that integral values will implicitly convert into Boolean values: an integral value of `0` converts to Boolean value `false`, and any other integral value converts to Boolean value `true`.

Similarly, pointers will also implicitly convert to Boolean values: a null pointer converts to Boolean value `false`, and a non-null pointer converts to Boolean value `true`. This allows us to skip explicitly testing for `nullptr` and just use the implicit conversion to Boolean to test whether a pointer is a null pointer. The following program is equivalent to the prior one:

```cpp
#include <iostream>

int main()
{
    int x { 5 };
    int* ptr { &x };

    // pointers convert to Boolean false if they are null, and Boolean true if they are non-null
    if (ptr) // implicit conversion to Boolean
        std::cout << "ptr is non-null\n";
    else
        std::cout << "ptr is null\n";

    int* nullPtr {};
    std::cout << "nullPtr is " << (nullPtr ? "non-null\n" : "null\n"); // implicit conversion to Boolean

    return 0;
}
```

**Warning**

Conditionals can only be used to differentiate null pointers from non-null pointers. There is no convenient way to determine whether a non-null pointer is pointing to a valid object or dangling (pointing to an invalid object).

------

#### Use nullptr to avoid dangling pointers

Above, we mentioned that dereferencing a pointer that is either null or dangling will result in undefined behavior. Therefore, we need to ensure our code does not do either of these things.

We can easily avoid dereferencing a null pointer by using a conditional to ensure a pointer is non-null before trying to dereference it:

```cpp
// Assume ptr is some pointer that may or may not be a null pointer
if (ptr) // if ptr is not a null pointer
    std::cout << *ptr << '\n'; // okay to dereference
else
    // do something else that doesn't involve dereferencing ptr (print an error message, do nothing at all, etc...)
```

But what about dangling pointers? Because there is no way to detect whether a pointer is dangling, we need to avoid having any dangling pointers in our program in the first place. We do that by ensuring that any pointer that is not pointing at a valid object is set to `nullptr`.

That way, before dereferencing a pointer, we only need to test whether it is null -- if it is non-null, we assume the pointer is not dangling.

**Best practice**

A pointer should either hold the address of a valid object, or be set to nullptr. That way we only need to test pointers for null, and can assume any non-null pointer is valid.

Unfortunately, avoiding dangling pointers isn’t always easy: when an object is destroyed, any pointers to that object will be left dangling. Such pointers are *not* nulled automatically! It is the programmer’s responsibility to ensure that all pointers to an object that has just been destroyed are properly set to `nullptr`.

**Warning**

When an object is destroyed, any pointers to the destroyed object will be left dangling (they will not be automatically set to `nullptr`). It is your responsibility to detect these cases and ensure those pointers are subsequently set to `nullptr`.

------

#### Legacy null pointer literals: 0 and NULL

In older code, you may see two other literal values used instead of `nullptr`.

The first is the literal `0`. In the context of a pointer, the literal `0` is specially defined to mean a null value, and is the only time you can assign an integral literal to a pointer.

```cpp
int main()
{
    float* ptr { 0 };  // ptr is now a null pointer (for example only, don't do this)

    float* ptr2; // ptr2 is uninitialized
    ptr2 = 0; // ptr2 is now a null pointer (for example only, don't do this)

    return 0;
}
```

**As an aside…**

On modern architectures, the address `0` is typically used to represent a null pointer. However, this value is not guaranteed by the C++ standard, and some architectures use other values. The literal `0`, when used in the context of a null pointer, will be translated into whatever address the architecture uses to represent a null pointer.

Additionally, there is a preprocessor macro named `NULL` (defined in the <cstddef> header). This macro is inherited from C, where it is commonly used to indicate a null pointer.

```cpp
#include <cstddef> // for NULL

int main()
{
    double* ptr { NULL }; // ptr is a null pointer

    double* ptr2; // ptr2 is uninitialized
    ptr2 = NULL; // ptr2 is now a null pointer
}
```

Both `0` and `NULL` should be avoided in modern C++ (use `nullptr` instead). We discuss why in lesson [9.10 -- Pass by address (part 2)](https://www.learncpp.com/cpp-tutorial/pass-by-address-part-2/).

------

#### Favor references over pointers whenever possible

Pointers and references both give us the ability to access some other object indirectly.

Pointers have the additional abilities of being able to change what they are pointing at, and to be pointed at null. However, these pointer abilities are also inherently dangerous: A null pointer runs the risk of being dereferenced, and the ability to change what a pointer is pointing at can make creating dangling pointers easier:

```cpp
int main()
{
    int* ptr { };

    {
        int x{ 5 };
        ptr = &x; // set the pointer to an object that will be destroyed (not possible with a reference)
    } // ptr is now dangling

    return 0;
}
```

Since references can’t be bound to null, we don’t have to worry about null references. And because references must be bound to a valid object upon creation and then can not be reseated, dangling references are harder to create.

Because they are safer, references should be favored over pointers, unless the additional capabilities provided by pointers are required.

**Best practice**

Favor references over pointers unless the additional capabilities provided by pointers are needed.

------

#### Quiz time

**Question #1**

a) Can we determine whether a pointer is a null pointer or not? If so, how?

Yes, we can use a conditional (if statement or conditional operator) on the pointer. A pointer will convert to Boolean `false` if it is a null pointer, and `true` otherwise.

b) Can we determine whether a non-null pointer is valid or dangling? If so, how?

There is no easy way to determine this.

------

**Question #2**

For each subitem, answer “yes”, “no”, or “possibly” to whether the action described will result in undefined behavior (immediately). If the answer is “possibly”, clarify when.

a) Assigning a new address to a pointer

No

b) Assigning nullptr to a pointer

No

c) Dereferencing a pointer to a valid object

No

d) Dereferencing a dangling pointer

Yes

e) Dereferencing a null pointer

Yes

f) Dereferencing a non-null pointer

Possibly, if the pointer is dangling

------

**Question #3**

Why should we set pointers that aren’t pointing to a valid object to ‘nullptr’?

We can not determine whether a non-null pointer is valid or dangling, and accessing a dangling pointer will result in undefined behavior. Therefore, we need to ensure that we do not have any dangling pointers in our program.

If we ensure all pointers are either pointing to valid objects or set to `nullptr`, then we can use a conditional to test for null to ensure we don’t dereference a null pointer, and assume all non-null pointers are pointing to valid objects.

------

### 9.8 — Pointers and const

Consider the following code snippet:

```cpp
int main()
{
    int x { 5 };
    int* ptr { &x }; // ptr is a normal (non-const) pointer

    int y { 6 };
    ptr = &y; // we can point at another value

    *ptr = 7; // we can change the value at the address being held

    return 0;
}
```

With normal (non-const) pointers, we can change both what the pointer is pointing at (by assigning the pointer a new address to hold) or change the value at the address being held (by assigning a new value to the dereferenced pointer).

However, what happens if the value we want to point at is const?

```cpp
int main()
{
    const int x { 5 }; // x is now const
    int* ptr { &x };   // compile error: cannot convert from const int* to int*

    return 0;
}
```

The above snippet won’t compile -- we can’t set a normal pointer to point at a const variable. This makes sense: a const variable is one whose value cannot be changed. Allowing the programmer to set a non-const pointer to a const value would allow the programmer to dereference the pointer and change the value. That would violate the const-ness of the variable.

------

#### Pointer to const value

A **pointer to a const value** (sometimes called a `pointer to const` for short) is a (non-const) pointer that points to a constant value.

To declare a pointer to a const value, use the `const` keyword before the pointer’s data type:

```cpp
int main()
{
    const int x{ 5 };
    const int* ptr { &x }; // okay: ptr is pointing to a "const int"

    *ptr = 6; // not allowed: we can't change a const value

    return 0;
}
```

In the above example, `ptr` points to a `const int`. Because the data type being pointed to is const, the value being pointed to can’t be changed.

However, because a pointer to const is not const itself (it just points to a const value), we can change what the pointer is pointing at by assigning the pointer a new address:

```cpp
int main()
{
    const int x{ 5 };
    const int* ptr { &x }; // ptr points to const int x

    const int y{ 6 };
    ptr = &y; // okay: ptr now points at const int y

    return 0;
}
```

Just like a reference to const, a pointer to const can point to non-const variables too. A pointer to const treats the value being pointed to as constant, regardless of whether the object at that address was initially defined as const or not:

```cpp
int main()
{
    int x{ 5 }; // non-const
    const int* ptr { &x }; // ptr points to a "const int"

    *ptr = 6;  // not allowed: ptr points to a "const int" so we can't change the value through ptr
    x = 6; // allowed: the value is still non-const when accessed through non-const identifier x

    return 0;
}
```

------

#### Const pointers

We can also make a pointer itself constant. A **const pointer** is a pointer whose address can not be changed after initialization.

To declare a const pointer, use the `const` keyword after the asterisk in the pointer declaration:

```cpp
int main()
{
    int x{ 5 };
    int* const ptr { &x }; // const after the asterisk means this is a const pointer

    return 0;
}
```

In the above case, `ptr` is a const pointer to a (non-const) int value.

Just like a normal const variable, a const pointer must be initialized upon definition, and this value can’t be changed via assignment:

```cpp
int main()
{
    int x{ 5 };
    int y{ 6 };

    int* const ptr { &x }; // okay: the const pointer is initialized to the address of x
    ptr = &y; // error: once initialized, a const pointer can not be changed.

    return 0;
}
```

However, because the *value* being pointed to is non-const, it is possible to change the value being pointed to via dereferencing the const pointer:

```cpp
int main()
{
    int x{ 5 };
    int* const ptr { &x }; // ptr will always point to x

    *ptr = 6; // okay: the value being pointed to is non-const

    return 0;
}
```

------

#### Const pointer to a const value

Finally, it is possible to declare a **const pointer to a const value** by using the `const` keyword both before the type and after the asterisk:

```cpp
int main()
{
    int value { 5 };
    const int* const ptr { &value }; // a const pointer to a const value

    return 0;
}
```

A const pointer to a const value can not have its address changed, nor can the value it is pointing to be changed through the pointer. It can only be dereferenced to get the value it is pointing at.

------

#### Pointer and const recap

To summarize, you only need to remember 4 rules, and they are pretty logical:

- A non-const pointer can be assigned another address to change what it is pointing at.
- A const pointer always points to the same address, and this address can not be changed.

- A pointer to a non-const value can change the value it is pointing to. These can not point to a const value.
- A pointer to a const value treats the value as const when accessed through the pointer, and thus can not change the value it is pointing to. These can be pointed to const or non-const l-values (but not r-values, which don’t have an address).

Keeping the declaration syntax straight can be a bit challenging:

- The pointer’s type defines the type of the object being pointed at. So a `const` in the type means the pointer is pointing at a const value.
- A `const` after the asterisk means the pointer itself is const and it can not be assigned a new address.

```cpp
int main()
{
    int value { 5 };

    int* ptr0 { &value };             // ptr0 points to an "int" and is not const itself, so this is a normal pointer.
    const int* ptr1 { &value };       // ptr1 points to a "const int", but is not const itself, so this is a pointer to a const value.
    int* const ptr2 { &value };       // ptr2 points to an "int", but is const itself, so this is a const pointer (to a non-const value).
    const int* const ptr3 { &value }; // ptr3 points to an "const int", and it is const itself, so this is a const pointer to a const value.

    return 0;
}
```

------

### 9.9 — Pass by address

In prior lessons, we’ve covered two different ways to pass an argument to a function: pass by value ([2.4 -- Introduction to function parameters and arguments](https://www.learncpp.com/cpp-tutorial/introduction-to-function-parameters-and-arguments/)) and pass by reference ([9.5 -- Pass by lvalue reference](https://www.learncpp.com/cpp-tutorial/pass-by-lvalue-reference/)).

Here’s a sample program that shows a `std::string` object being passed by value and by reference:

```cpp
#include <iostream>
#include <string>

void printByValue(std::string val) // The function parameter is a copy of str
{
    std::cout << val << '\n'; // print the value via the copy
}

void printByReference(const std::string& ref) // The function parameter is a reference that binds to str
{
    std::cout << ref << '\n'; // print the value via the reference
}

int main()
{
    std::string str{ "Hello, world!" };

    printByValue(str); // pass str by value, makes a copy of str
    printByReference(str); // pass str by reference, does not make a copy of str

    return 0;
}
```

When we pass argument `str` by value, the function parameter `val` receives a copy of the argument. Because the parameter is a copy of the argument, any changes to the `val` are made to the copy, not the original argument.

When we pass argument `str` by reference, the reference parameter `ref` is bound to the actual argument. This avoids making a copy of the argument. Because our reference parameter is const, we are not allowed to change `ref`. But if `ref` were non-const, any changes we made to `ref` would change `str`.

In both cases, the caller is providing the actual object (`str`) to be passed as an argument to the function call.

------

#### Pass by address

C++ provides a third way to pass values to a function, called pass by address. With **pass by address**, instead of providing an object as an argument, the caller provides an object’s *address* (via a pointer). This pointer (holding the address of the object) is copied into a pointer parameter of the called function (which now also holds the address of the object). The function can then dereference that pointer to access the object whose address was passed.

Here’s a version of the above program that adds a pass by address variant:

```cpp
#include <iostream>
#include <string>

void printByValue(std::string val) // The function parameter is a copy of str
{
    std::cout << val << '\n'; // print the value via the copy
}

void printByReference(const std::string& ref) // The function parameter is a reference that binds to str
{
    std::cout << ref << '\n'; // print the value via the reference
}

void printByAddress(const std::string* ptr) // The function parameter is a pointer that holds the address of str
{
    std::cout << *ptr << '\n'; // print the value via the dereferenced pointer
}

int main()
{
    std::string str{ "Hello, world!" };

    printByValue(str); // pass str by value, makes a copy of str
    printByReference(str); // pass str by reference, does not make a copy of str
    printByAddress(&str); // pass str by address, does not make a copy of str

    return 0;
}
```

Note how similar all three of these versions are. Let’s explore the pass by address version in more detail.

First, because we want our `printByAddress()` function to use pass by address, we’ve made our function parameter a pointer named `ptr`. Since `printByAddress()` will use `ptr` in a read-only manner, `ptr` is a pointer to a const value.

```cpp
void printByAddress(const std::string* ptr)
{
    std::cout << *ptr << '\n'; // print the value via the dereferenced pointer
}
```

Inside the `printByAddress()` function, we dereference `ptr` parameter to access the value of the object being pointed to.

Second, when the function is called, we can’t just pass in the `str` object -- we need to pass in the address of `str`. The easiest way to do that is to use the address-of operator (&) to get a pointer holding the address of `str`:

```cpp
printByAddress(&str); // use address-of operator (&) to get pointer holding address of str
```

When this call is executed, `&str` will create a pointer holding the address of `str`. This address is then copied into function parameter `ptr` as part of the function call. Because `ptr` now holds the address of `str`, when the function dereferences `ptr`, it will get the value of `str`, which the function prints to the console.

That’s it.

Although we use the address-of operator in the above example to get the address of `str`, if we already had a pointer variable holding the address of `str`, we could use that instead:

```cpp
int main()
{
    std::string str{ "Hello, world!" };

    printByValue(str); // pass str by value, makes a copy of str
    printByReference(str); // pass str by reference, does not make a copy of str
    printByAddress(&str); // pass str by address, does not make a copy of str

    std::string* ptr { &str }; // define a pointer variable holding the address of str
    printByAddress(ptr); // pass str by address, does not make a copy of str

    return 0;
}
```

------

#### Pass by address does not make a copy of the object being pointed to

Consider the following statements:

```cpp
std::string str{ "Hello, world!" };
printByAddress(&str); // use address-of operator (&) to get pointer holding address of str
```

As we noted in [9.5 -- Pass by lvalue reference](https://www.learncpp.com/cpp-tutorial/pass-by-lvalue-reference/), copying a `std::string` is expensive, so that’s something we want to avoid. When we pass a `std::string` by address, we’re not copying the actual `std::string` object -- we’re just copying the pointer (holding the address of the object) from the caller to the called function. Since an address is typically only 4 or 8 bytes, a pointer is only 4 or 8 bytes, so copying a pointer is always fast.

Thus, just like pass by reference, pass by address is fast, and avoids making a copy of the argument object.

------

#### Pass by address allows the function to modify the argument’s value

When we pass an object by address, the function receives the address of the passed object, which it can access via dereferencing. Because this is the address of the actual argument object being passed (not a copy of the object), if the function parameter is a pointer to non-const, the function can modify the argument via the pointer parameter:

```cpp
#include <iostream>

void changeValue(int* ptr) // note: ptr is a pointer to non-const in this example
{
    *ptr = 6; // change the value to 6
}

int main()
{
    int x{ 5 };

    std::cout << "x = " << x << '\n';

    changeValue(&x); // we're passing the address of x to the function

    std::cout << "x = " << x << '\n';

    return 0;
}
```

This prints:

```
x = 5
x = 6
```

As you can see, the argument is modified and this modification persists even after `changeValue()` has finished running.

If a function is not supposed to modify the object being passed in, the function parameter can be made a pointer to const:

```cpp
void changeValue(const int* ptr) // note: ptr is now a pointer to a const
{
    *ptr = 6; // error: can not change const value
}
```

------

#### Null checking

Now consider this fairly innocent looking program:

```cpp
#include <iostream>

void print(int* ptr)
{
	std::cout << *ptr;
}

int main()
{
	int x{ 5 };
	print(&x);

	int* myptr {};
	print(myptr);

	return 0;
}
```

When this program is run, it will print the value `5` and then most likely crash.

In the call to `print(myptr)`, `myptr` is a null pointer, so function parameter `ptr` will also be a null pointer. When this null pointer is dereferenced in the body of the function, undefined behavior results.

When passing a parameter by address, care should be taken to ensure the pointer is not a null pointer before you dereference the value. One way to do that is to use a conditional statement:

```cpp
#include <iostream>

void print(int* ptr)
{
    if (ptr) // if ptr is not a null pointer
    {
        std::cout << *ptr;
    }
}

int main()
{
	int x{ 5 };

	print(&x);
	print(nullptr);

	return 0;
}
```

In the above program, we’re testing `ptr` to ensure it is not null before we dereference it. While this is fine for such a simple function, in more complicated functions this can result in redundant logic (testing if ptr is not null multiple times) or nesting of the primary logic of the function (if contained in a block).

In most cases, it is more effective to do the opposite: test whether the function parameter is null as a precondition ([7.18 -- Assert and static_assert](https://www.learncpp.com/cpp-tutorial/assert-and-static_assert/)) and handle the negative case immediately:

```cpp
#include <iostream>

void print(int* ptr)
{
    if (!ptr) // if ptr is a null pointer, early return back to the caller
        return;

    // if we reached this point, we can assume ptr is valid
    // so no more testing or nesting required

    std::cout << *ptr;
}

int main()
{
	int x{ 5 };

	print(&x);
	print(nullptr);

	return 0;
}
```

If a null pointer should never be passed to the function, an `assert` (which we covered in lesson [7.18 -- Assert and static_assert](https://www.learncpp.com/cpp-tutorial/assert-and-static_assert/)) can be used instead (or also) (as asserts are intended to document things that should never happen):

```cpp
#include <iostream>
#include <cassert>

void print(const int* ptr) // now a pointer to a const int
{
	assert(ptr); // fail the program in debug mode if a null pointer is passed (since this should never happen)

	// (optionally) handle this as an error case in production mode so we don't crash if it does happen
	if (!ptr)
		return;

	std::cout << *ptr;
}

int main()
{
	int x{ 5 };

	print(&x);
	print(nullptr);

	return 0;
}
```

------

#### Prefer pass by (const) reference

Note that function `print()` in the example above doesn’t handle null values very well -- it effectively just aborts the function. Given this, why allow a user to pass in a null value at all? Pass by reference has the same benefits as pass by address without the risk of inadvertently dereferencing a null pointer.

Pass by const reference has a few other advantages over pass by address.

First, because an object being passed by address must have an address, only lvalues can be passed by address (as rvalues don’t have addresses). Pass by const reference is more flexible, as it can accept lvalues and rvalues:

```cpp
#include <iostream>
#include <string>

void printByValue(int val) // The function parameter is a copy of the argument
{
    std::cout << val << '\n'; // print the value via the copy
}

void printByReference(const int& ref) // The function parameter is a reference that binds to the argument
{
    std::cout << ref << '\n'; // print the value via the reference
}

void printByAddress(const int* ptr) // The function parameter is a pointer that holds the address of the argument
{
    std::cout << *ptr << '\n'; // print the value via the dereferenced pointer
}

int main()
{
    printByValue(5);     // valid (but makes a copy)
    printByReference(5); // valid (because the parameter is a const reference)
    printByAddress(&5);  // error: can't take address of r-value

    return 0;
}
```

Second, the syntax for pass by reference is natural, as we can just pass in literals or objects. With pass by address, our code ends up littered with ampersands (&) and asterisks (*).

In modern C++, most things that can be done with pass by address are better accomplished through other methods. Follow this common maxim: “Pass by reference when you can, pass by address when you must”.

**Best practice**

Prefer pass by reference to pass by address unless you have a specific reason to use pass by address.

------

### 9.10 — Pass by address (part 2)

This lesson is a continuation of [9.9 -- Pass by address](https://www.learncpp.com/cpp-tutorial/pass-by-address/).

------

#### Pass by address for “optional” arguments

One of the more common uses for pass by address is to allow a function to accept an “optional” argument. This is easier to illustrate by example than to describe:

```cpp
#include <iostream>
#include <string>

void greet(std::string* name=nullptr)
{
    std::cout << "Hello ";
    std::cout << (name ? *name : "guest") << '\n';
}

int main()
{
    greet(); // we don't know who the user is yet

    std::string joe{ "Joe" };
    greet(&joe); // we know the user is joe

    return 0;
}
```

This example prints:

```
Hello guest
Hello Joe
```

In this program, the `greet()` function has one parameter that is passed by address and defaulted to `nullptr`. Inside `main()`, we call this function twice. The first call, we don’t know who the user is, so we call `greet()` without an argument. The `name` parameter defaults to `nullptr`, and the greet function substitutes in the name “guest”. For the second call, we now have a valid user, so we call `greet(&joe)`. The `name` parameter receives the address of `joe`, and can use it to print the name “Joe”.

However, in many cases, function overloading is a better alternative to achieve the same result:

```cpp
#include <iostream>
#include <string>
#include <string_view>

void greet(std::string_view name)
{
    std::cout << "Hello " << name << '\n';
}

void greet()
{
    greet("guest");
}

int main()
{
    greet(); // we don't know who the user is yet

    std::string joe{ "Joe" };
    greet(joe); // we know the user is joe

    return 0;
}
```

This has a number of advantages: we no longer have to worry about null dereferences, and we could pass in a string literal if we wanted.

------

#### Changing what a pointer parameter points at

When we pass an address to a function, that address is copied from the argument into the pointer parameter (which is fine, because copying an address is fast). Now consider the following program:

```cpp
#include <iostream>

// [[maybe_unused]] gets rid of compiler warnings about ptr2 being set but not used
void nullify([[maybe_unused]] int* ptr2)
{
    ptr2 = nullptr; // Make the function parameter a null pointer
}

int main()
{
    int x{ 5 };
    int* ptr{ &x }; // ptr points to x

    std::cout << "ptr is " << (ptr ? "non-null\n" : "null\n");

    nullify(ptr);

    std::cout << "ptr is " << (ptr ? "non-null\n" : "null\n");
    return 0;
}
```

This program prints:

```
ptr is non-null
ptr is non-null
```

As you can see, changing the address held by the pointer parameter had no impact on the address held by the argument (`ptr` still points at `x`). When function `nullify()` is called, `ptr2` receives a copy of the address passed in (in this case, the address held by `ptr`, which is the address of `x`). When the function changes what `ptr2` points at, this only affects the copy held by `ptr2`.

So what if we want to allow a function to change what a pointer argument points to?

------

#### Pass by address… by reference?

Yup, it’s a thing. Just like we can pass a normal variable by reference, we can also pass pointers by reference. Here’s the same program as above with `ptr2` changed to be a reference to an address:

```cpp
#include <iostream>

void nullify(int*& refptr) // refptr is now a reference to a pointer
{
    refptr = nullptr; // Make the function parameter a null pointer
}

int main()
{
    int x{ 5 };
    int* ptr{ &x }; // ptr points to x

    std::cout << "ptr is " << (ptr ? "non-null\n" : "null\n");

    nullify(ptr);

    std::cout << "ptr is " << (ptr ? "non-null\n" : "null\n");
    return 0;
}
```

This program prints:

```
ptr is non-null
ptr is null
```

Because `refptr` is now a reference to a pointer, when `ptr` is passed as an argument, `refptr` is bound to `ptr`. This means any changes to `refptr` are made to `ptr`.

**As an aside…**

Because references to pointers are fairly uncommon, it can be easy to mix up the syntax (is it `int*&` or `int&*`?). The good news is that if you do it backwards, the compiler will error because you can’t have a pointer to a reference (because pointers must hold the address of an object, and references aren’t objects). Then you can switch it around.

------

#### Why using `0` or `NULL` is no longer preferred (optional)

In this subsection, we’ll explain why using `0` or `NULL` is no longer preferred.

The literal `0` can be interpreted as either an integer literal, or as a null pointer literal. In certain cases, it can be ambiguous which one we intend -- and in some of those cases, the compiler may assume we mean one when we mean the other -- with unintended consequences to the behavior of our program.

The definition of preprocessor macro `NULL` is not defined by the language standard. It can be defined as `0`, `0L`, `((void*)0)`, or something else entirely.

In lesson [8.10 -- Introduction to function overloading](https://www.learncpp.com/cpp-tutorial/introduction-to-function-overloading/), we discussed that functions can be overloaded (multiple functions can have the same name, so long as they can be differentiated by the number or type of parameters). The compiler can figure out which overloaded function you desire by the arguments passed in as part of the function call.

When using `0` or `NULL`, this can cause problems:

```cpp
#include <iostream>
#include <cstddef> // for NULL

void print(int x) // this function accepts an integer
{
	std::cout << "print(int): " << x << '\n';
}

void print(int* ptr) // this function accepts an integer pointer
{
	std::cout << "print(int*): " << (ptr ? "non-null\n" : "null\n");
}

int main()
{
	int x{ 5 };
	int* ptr{ &x };

	print(ptr);  // always calls print(int*) because ptr has type int* (good)
	print(0);    // always calls print(int) because 0 is an integer literal (hopefully this is what we expected)

	print(NULL); // this statement could do any of the following:
	// call print(int) (Visual Studio does this)
	// call print(int*)
	// result in an ambiguous function call compilation error (gcc and Clang do this)

	print(nullptr); // always calls print(int*)

	return 0;
}
```

On the author’s machine (using Visual Studio), this prints:

```
print(int*): non-null
print(int): 0
print(int): 0
print(int*): null
```

When passing integer value `0` as a parameter, the compiler will prefer `print(int)` over `print(int*)`. This can lead to unexpected results when we intended `print(int*)` to be called with a null pointer argument.

In the case where `NULL` is defined as value `0`, `print(NULL)` will also call `print(int)`, not `print(int*)` like you might expect for a null pointer literal. In cases where `NULL` is not defined as `0`, other behavior might result, like a call to `print(int*)` or a compilation error.

Using `nullptr` removes this ambiguity (it will always call `print(int*)`), since `nullptr` will only match a pointer type.

------

#### std::nullptr_t (optional)

Since `nullptr` can be differentiated from integer values in function overloads, it must have a different type. So what type is `nullptr`? The answer is that `nullptr` has type `std::nullptr_t` (defined in header <cstddef>). `std::nullptr_t` can only hold one value: `nullptr`! While this may seem kind of silly, it’s useful in one situation. If we want to write a function that accepts only a `nullptr` literal argument, we can make the parameter a `std::nullptr_t`.

```cpp
#include <iostream>
#include <cstddef> // for std::nullptr_t

void print(std::nullptr_t)
{
    std::cout << "in print(std::nullptr_t)\n";
}

void print(int*)
{
    std::cout << "in print(int*)\n";
}

int main()
{
    print(nullptr); // calls print(std::nullptr_t)

    int x { 5 };
    int* ptr { &x };

    print(ptr); // calls print(int*)

    ptr = nullptr;
    print(ptr); // calls print(int*) (since ptr has type int*)

    return 0;
}
```

In the above example, the function call `print(nullptr)` resolves to the function `print(std::nullptr_t)` over `print(int*)` because it doesn’t require a conversion.

The one case that might be a little confusing is when we call `print(ptr)` when `ptr` is holding the value `nullptr`. Remember that function overloading matches on types, not values, and `ptr` has type `int*`. Therefore, `print(int*)` will be matched. `print(std::nullptr_t)` isn’t even in consideration in this case, as pointer types will not implicitly convert to a `std::nullptr_t`.

You probably won’t ever need to use this, but it’s good to know, just in case.

------

#### There is only pass by value

Now that you understand the basic differences between passing by reference, address, and value, let’s get reductionist for a moment. :)

While the compiler can often optimize references away entirely, there are cases where this is not possible and a reference is actually needed. References are normally implemented by the compiler using pointers. This means that behind the scenes, pass by reference is essentially just a pass by address (with access to the reference doing an implicit dereference).

And in the previous lesson, we mentioned that pass by address just copies an address from the caller to the called function -- which is just passing an address by value.

Therefore, we can conclude that C++ really passes everything by value! The properties of pass by address (and reference) come solely from the fact that we can dereference the passed address to change the argument, which we can not do with a normal value parameter!

------

### 9.11 — Return by reference and return by address

In previous lessons, we discussed that when passing an argument by value, a copy of the argument is made into the function parameter. For fundamental types (which are cheap to copy), this is fine. But copying is typically expensive for class types (such as `std::string`). We can avoid making an expensive copy by utilizing passing by (const) reference (or pass by address) instead.

We encounter a similar situation when returning by value: a copy of the return value is passed back to the caller. If the return type of the function is a class type, this can be expensive.

```cpp
std::string returnByValue(); // returns a copy of a std::string (expensive)
```

------

#### Return by reference

In cases where we’re passing a class type back to the caller, we may (or may not) want to return by reference instead. **Return by reference** returns a reference that is bound to the object being returned, which avoids making a copy of the return value. To return by reference, we simply define the return value of the function to be a reference type:

```cpp
std::string&       returnByReference(); // returns a reference to an existing std::string (cheap)
const std::string& returnByReferenceToConst(); // returns a const reference to an existing std::string (cheap)
```

Here is an academic program to demonstrate the mechanics of return by reference:

```cpp
#include <iostream>
#include <string>

const std::string& getProgramName() // returns a const reference
{
    static const std::string s_programName { "Calculator" }; // has static duration, destroyed at end of program

    return s_programName;
}

int main()
{
    std::cout << "This program is named " << getProgramName();

    return 0;
}
```

This program prints:

```
This program is named Calculator
```

Because `getProgramName()` returns a const reference, when the line `return s_programName` is executed, `getProgramName()` will return a const reference to `s_programName` (thus avoiding making a copy). That const reference can then be used by the caller to access the value of `s_programName`, which is printed.

------

#### The object being returned by reference must exist after the function returns

Using return by reference has one major caveat: the programmer *must* be sure that the object being referenced outlives the function returning the reference. Otherwise, the reference being returned will be left dangling (referencing an object that has been destroyed), and use of that reference will result in undefined behavior.

In the program above, because `s_programName` has static duration, `s_programName` will exist until the end of the program. When `main()` accesses the returned reference, it is actually accessing `s_programName`, which is fine, because `s_programName` won’t be destroyed until later.

Now let’s modify the above program to show what happens in the case where our function returns a dangling reference:

```cpp
#include <iostream>
#include <string>

const std::string& getProgramName()
{
    const std::string programName { "Calculator" }; // now a local variable, destroyed when function ends

    return programName;
}

int main()
{
    std::cout << "This program is named " << getProgramName();

    return 0;
}
```

The result of this program is undefined. When `getProgramName()` returns, a reference bound to local variable `programName` is returned. Then, because `programName` is a local variable with automatic duration, `programName` is destroyed at the end of the function. That means the returned reference is now dangling, and use of `programName` in the `main()` function results in undefined behavior.

Modern compilers will produce a warning or error if you try to return a local variable by reference (so the above program may not even compile), but compilers sometimes have trouble detecting more complicated cases.

**Warning**

Objects returned by reference must live beyond the scope of the function returning the reference, or a dangling reference will result. Never return a local variable by reference.

------

#### Don’t return non-const local static variables by reference

In the original example above, we returned a const local static variable by reference to illustrate the mechanics of return by reference in a simple way. However, returning non-const static variables by reference is fairly non-idiomatic, and should generally be avoided. Here’s a simplified example that illustrates one such problem that can occur:

```cpp
#include <iostream>
#include <string>

const int& getNextId()
{
    static int s_x{ 0 }; // note: variable is non-const
    ++s_x; // generate the next id
    return s_x; // and return a reference to it
}

int main()
{
    const int& id1 { getNextId() }; // id1 is a reference
    const int& id2 { getNextId() }; // id2 is a reference

    std::cout << id1 << id2 << '\n';

    return 0;
}
```

This program prints:

```
22
```

This happens because `id1` and `id2` are referencing the same object (the static variable `s_x`), so when anything (e.g. `getNextId()`) modifies that value, all references are now accessing the modified value. Another issue that commonly occurs with programs that return a static local by const reference is that there is no standardized way to reset `s_x` back to the default state. Such programs must either use a non-idiomatic solution (e.g. a reset parameter), or can only be reset by quitting and restarting the program.

While the above example is a bit silly, there are permutations of the above that programmers sometimes try for optimization purposes, and then their programs don’t work as expected.

**Best practice**

Avoid returning references to non-const local static variables.

Returning a const reference to a *const* local static variable is sometimes done if the local variable being returned by reference is expensive to create (so we don’t have to recreate the variable every function call). But this is rare.

Returning a const reference to a *const* global variable is also sometimes done as a way to encapsulate access to a global variable. We discuss this in lesson [6.8 -- Why (non-const) global variables are evil](https://www.learncpp.com/cpp-tutorial/why-non-const-global-variables-are-evil/). When used intentionally and carefully, this is also okay.

------

#### Assigning/initializing a normal variable with a returned reference makes a copy

If a function returns a reference, and that reference is used to initialize or assign to a non-reference variable, the return value will be copied (as if it had been returned by value).

```cpp
#include <iostream>
#include <string>

const int& getNextId()
{
    static int s_x{ 0 };
    ++s_x;
    return s_x;
}

int main()
{
    const int id1 { getNextId() }; // id1 is a normal variable now and receives a copy of the value returned by reference from getNextId()
    const int id2 { getNextId() }; // id2 is a normal variable now and receives a copy of the value returned by reference from getNextId()

    std::cout << id1 << id2 << '\n';

    return 0;
}
```

In the above example, `getNextId()` is returning a reference, but `id1` and `id2` are non-reference variables. In such a case, the value of the returned reference is copied into the normal variable. Thus, this program prints:

```
12
```

Of course, this also defeats the purpose of returning a value by reference.

Also note that if a program returns a dangling reference, the reference is left dangling before the copy is made, which will lead to undefined behavior:

```cpp
#include <iostream>
#include <string>

const std::string& getProgramName() // will return a const reference
{
    const std::string programName{ "Calculator" };

    return programName;
}

int main()
{
    std::string name { getProgramName() }; // makes a copy of a dangling reference
    std::cout << "This program is named " << name << '\n'; // undefined behavior

    return 0;
}
```

------

#### It’s okay to return reference parameters by reference

There are quite a few cases where returning objects by reference makes sense, and we’ll encounter many of those in future lessons. However, there is one useful example that we can show now.

If a parameter is passed into a function by reference, it’s safe to return that parameter by reference. This makes sense: in order to pass an argument to a function, the argument must exist in the scope of the caller. When the called function returns, that object must still exist in the scope of the caller.

Here is a simple example of such a function:

```cpp
#include <iostream>
#include <string>

// Takes two std::string objects, returns the one that comes first alphabetically
const std::string& firstAlphabetical(const std::string& a, const std::string& b)
{
	return (a < b) ? a : b; // We can use operator< on std::string to determine which comes first alphabetically
}

int main()
{
	std::string hello { "Hello" };
	std::string world { "World" };

	std::cout << firstAlphabetical(hello, world) << '\n';

	return 0;
}
```

This prints:

```
Hello
```

In the above function, the caller passes in two std::string objects by const reference, and whichever of these strings comes first alphabetically is passed back by const reference. If we had used pass by value and return by value, we would have made up to 3 copies of std::string (one for each parameter, one for the return value). By using pass by reference/return by reference, we can avoid those copies.

------

#### The caller can modify values through the reference

When an argument is passed to a function by non-const reference, the function can use the reference to modify the value of the argument.

Similarly, when a non-const reference is returned from a function, the caller can use the reference to modify the value being returned.

Here’s an illustrative example:

```cpp
#include <iostream>

// takes two integers by non-const reference, and returns the greater by reference
int& max(int& x, int& y)
{
    return (x > y) ? x : y;
}

int main()
{
    int a{ 5 };
    int b{ 6 };

    max(a, b) = 7; // sets the greater of a or b to 7

    std::cout << a << b << '\n';

    return 0;
}
```

In the above program, `max(a, b)` calls the `max()` function with `a` and `b` as arguments. Reference parameter `x` binds to argument `a`, and reference parameter `y` binds to argument `b`. The function then determines which of `x` (`5`) and `y` (`6`) is greater. In this case, that’s `y`, so the function returns `y` (which is still bound to `b`) back to the caller. The caller then assigns the value `7` to this returned reference.

Therefore, the expression `max(a, b) = 7` effectively resolves to `b = 7`.

This prints:

```
57
```

------

#### Return by address

**Return by address** works almost identically to return by reference, except a pointer to an object is returned instead of a reference to an object. Return by address has the same primary caveat as return by reference -- the object being returned by address must outlive the scope of the function returning the address, otherwise the caller will receive a dangling pointer.

The major advantage of return by address over return by reference is that we can have the function return `nullptr` if there is no valid object to return. For example, let’s say we have a list of students that we want to search. If we find the student we are looking for in the list, we can return a pointer to the object representing the matching student. If we don’t find any students matching, we can return `nullptr` to indicate a matching student object was not found.

The major disadvantage of return by address is that the caller has to remember to do a `nullptr` check before dereferencing the return value, otherwise a null pointer dereference may occur and undefined behavior will result. Because of this danger, return by reference should be preferred over return by address unless the ability to return “no object” is needed.

**Best practice**

Prefer return by reference over return by address unless the ability to return “no object” (using `nullptr`) is important.

------

### 9.12 — Type deduction with pointers, references, and const

In lesson [8.8 -- Type deduction for objects using the auto keyword](https://www.learncpp.com/cpp-tutorial/type-deduction-for-objects-using-the-auto-keyword/), we discussed how the `auto` keyword can be used to have the compiler deduce the type of a variable from the initializer:

```cpp
int getVal(); // some function that returns an int by value

int main()
{
    auto val { getVal() }; // val deduced as type int

    return 0;
}
```

We also noted that by default, type deduction will drop `const` qualifiers:

```cpp
const double foo()
{
    return 5.6;
}

int main()
{
    const double cd{ 7.8 };

    auto x{ cd };    // double (const dropped)
    auto y{ foo() }; // double (const dropped)

    return 0;
}
```

Const can be reapplied by adding the `const` qualifier in the definition:

```cpp
const double foo()
{
    return 5.6;
}

int main()
{
    const double cd{ 7.8 };

    const auto x{ cd };    // const double (const reapplied)
    const auto y{ foo() }; // const double (const reapplied)

    return 0;
}
```

------

#### Type deduction drops references

In addition to dropping const qualifiers, type deduction will also drop references:

```cpp
#include <string>

std::string& getRef(); // some function that returns a reference

int main()
{
    auto ref { getRef() }; // type deduced as std::string (not std::string&)

    return 0;
}
```

In the above example, variable `ref` is using type deduction. Although function `getRef()` returns a `std::string&`, the reference qualifier is dropped, so the type of `ref` is deduced as `std::string`.

Just like with the dropped `const` qualifier, if you want the deduced type to be a reference, you can reapply the reference at the point of definition:

```cpp
#include <string>

std::string& getRef(); // some function that returns a reference

int main()
{
    auto ref1 { getRef() };  // std::string (reference dropped)
    auto& ref2 { getRef() }; // std::string& (reference reapplied)

    return 0;
}
```

------

#### Top-level const and low-level const

A **top-level const** is a const qualifier that applies to an object itself. For example:

```cpp
const int x;    // this const applies to x, so it is top-level
int* const ptr; // this const applies to ptr, so it is top-level
```

In contrast, a **low-level const** is a const qualifier that applies to the object being referenced or pointed to:

```cpp
const int& ref; // this const applies to the object being referenced, so it is low-level
const int* ptr; // this const applies to the object being pointed to, so it is low-level
```

A reference to a const value is always a low-level const. A pointer can have a top-level, low-level, or both kinds of const:

```cpp
const int* const ptr; // the left const is low-level, the right const is top-level
```

When we say that type deduction drops const qualifiers, it only drops top-level consts. Low-level consts are not dropped. We’ll see examples of this in just a moment.

------

#### Type deduction and const references

If the initializer is a reference to const, the reference is dropped first (and then reapplied if applicable), and then any top-level const is dropped from the result.

```cpp
#include <string>

const std::string& getConstRef(); // some function that returns a reference to const

int main()
{
    auto ref1{ getConstRef() }; // std::string (reference dropped, then top-level const dropped from result)

    return 0;
}
```

In the above example, since `getConstRef()` returns a `const std::string&`, the reference is dropped first, leaving us with a `const std::string`. This const is now a top-level const, so it is also dropped, leaving the deduced type as `std::string`.

We can reapply either or both of these:

```cpp
#include <string>

const std::string& getConstRef(); // some function that returns a const reference

int main()
{
    auto ref1{ getConstRef() };        // std::string (top-level const and reference dropped)
    const auto ref2{ getConstRef() };  // const std::string (const reapplied, reference dropped)

    auto& ref3{ getConstRef() };       // const std::string& (reference reapplied, low-level const not dropped)
    const auto& ref4{ getConstRef() }; // const std::string& (reference and const reapplied)

    return 0;
}
```

We covered the case for `ref1` in the prior example. For `ref2`, this is similar to the `ref1` case, except we’re reapplying the `const` qualifier, so the deduced type is `const std::string`.

Things get more interesting with `ref3`. Normally the reference would be dropped, but since we’ve reapplied the reference, it is not dropped. That means the type is still `const std::string&`. And since this const is a low-level const, it is not dropped. Thus the deduced type is `const std::string&`.

The `ref4` case works similarly to `ref3`, except we’ve reapplied the `const` qualifier as well. Since the type is already deduced as a reference to const, us reapplying `const` here is redundant. That said, using `const` here makes it explicitly clear that our result will be const (whereas in the `ref3` case, the constness of the result is implicit and not obvious).

**Best practice**

If you want a const reference, reapply the `const` qualifier even when it’s not strictly necessary, as it makes your intent clear and helps prevent mistakes.

------

#### Type deduction and pointers

Unlike references, type deduction does not drop pointers:

```cpp
#include <string>

std::string* getPtr(); // some function that returns a pointer

int main()
{
    auto ptr1{ getPtr() }; // std::string*

    return 0;
}
```

We can also use an asterisk in conjunction with pointer type deduction:

```cpp
#include <string>

std::string* getPtr(); // some function that returns a pointer

int main()
{
    auto ptr1{ getPtr() };  // std::string*
    auto* ptr2{ getPtr() }; // std::string*

    return 0;
}
```

------

#### The difference between auto and auto* (optional reading)

When we use `auto` with a pointer type initializer, the type deduced for `auto` includes the pointer. So for `ptr1` above, the type substituted for `auto` is `std::string*`.

When we use `auto*` with a pointer type initializer, the type deduced for auto does *not* include the pointer -- the pointer is reapplied afterward after the type is deduced. So for `ptr2` above, the type substituted for `auto` is `std::string`, and then the pointer is reapplied.

In most cases, the practical effect is the same (`ptr1` and `ptr2` both deduce to `std::string*` in the above example).

However, there are a couple of difference between `auto` and `auto*` in practice. First, `auto*` must resolve to a pointer initializer, otherwise a compile error will result:

```cpp
#include <string>

std::string* getPtr(); // some function that returns a pointer

int main()
{
    auto ptr3{ *getPtr() };      // std::string (because we dereferenced getPtr())
    auto* ptr4{ *getPtr() };     // does not compile (initializer not a pointer)

    return 0;
}
```

This makes sense: in the `ptr4` case, `auto` deduces to `std::string`, then the pointer is reapplied. Thus `ptr4` has type `std::string*`, and we can’t initialize a `std::string*` with an initializer that is not a pointer.

Second, there are differences in how `auto` and `auto*` behave when we introduce `const` into the equation. We’ll cover this below.

------

#### Type deduction and const pointers (optional reading)

Since pointers aren’t dropped, we don’t have to worry about that. But with pointers, we have both the const pointer and the pointer to const cases to think about, and we also have `auto` vs `auto*`. Just like with references, only top-level const is dropped during pointer type deduction.

Let’s start with a simple case:

```cpp
#include <string>

std::string* getPtr(); // some function that returns a pointer

int main()
{
    const auto ptr1{ getPtr() };  // std::string* const
    auto const ptr2 { getPtr() }; // std::string* const

    const auto* ptr3{ getPtr() }; // const std::string*
    auto* const ptr4{ getPtr() }; // std::string* const

    return 0;
}
```

When we use either `auto const` or `const auto`, we’re saying, “make whatever the deduced type is const”. So in the case of `ptr1` and `ptr2`, the deduced type is `std::string*`, and then const is applied, making the final type `std::string* const`. This is similar to how `const int` and `int const` mean the same thing.

However, when we use `auto*`, the order of the const qualifier matters. A `const` on the left means “make the deduced pointer type a pointer to const”, whereas a `const` on the right means “make the deduced pointer type a const pointer”. Thus `ptr3` ends up as a pointer to const, and `ptr4` ends up as a const pointer.

Now let’s look at an example where the initializer is a const pointer to const.

```cpp
#include <string>

const std::string* const getConstPtr(); // some function that returns a const pointer to a const value

int main()
{
    auto ptr1{ getConstPtr() };  // const std::string*
    auto* ptr2{ getConstPtr() }; // const std::string*

    auto const ptr3{ getConstPtr() };  // const std::string* const
    const auto ptr4{ getConstPtr() };  // const std::string* const

    auto* const ptr5{ getConstPtr() }; // const std::string* const
    const auto* ptr6{ getConstPtr() }; // const std::string*

    const auto const ptr7{ getConstPtr() };  // error: const qualifer can not be applied twice
    const auto* const ptr8{ getConstPtr() }; // const std::string* const

    return 0;
}
```

The `ptr1` and `ptr2` cases are straightforward. The top-level const (the const on the pointer itself) is dropped. The low-level const on the object being pointed to is not dropped. So in both cases, the final type is `const std::string*`.

The `ptr3` and `ptr4` cases are also straightforward. The top-level const is dropped, but we’re reapplying it. The low-level const on the object being pointed to is not dropped. So in both cases, the final type is `const std::string* const`.

The `ptr5` and `ptr6` cases are analogous to the cases we showed in the prior example. In both cases, the top-level const is dropped. For `ptr5`, the `auto* const` reapplies the top-level const, so the final type is `const std::string* const`. For `ptr6`, the `const auto*` applies const to the type being pointed to (which in this case was already const), so the final type is `const std::string*`.

In the `ptr7` case, we’re applying the const qualifier twice, which is disallowed, and will cause a compile error.

And finally, in the `ptr8` case, we’re applying const on both sides of the pointer (which is allowed since `auto*` must be a pointer type), so the resulting types is `const std::string* const`.

**Best practice**

If you want a const pointer, reapply the `const` qualifier even when it’s not strictly necessary, as it makes your intent clear and helps prevent mistakes.

------

### 9.x — Chapter 9 summary and quiz

#### Quick review

**Compound data types** (also called **composite data type**) are data types that can be constructed from fundamental data types (or other compound data types).

The **value category** of an expression indicates whether an expression resolves to a value, a function, or an object of some kind.

An **lvalue** is an expression that evaluates to a function or an object that has an identity. An object or function with an **identity** has an identifier or an identifiable memory address. Lvalues come in two subtypes: **modifiable lvalues** are lvalues that can be modified, and **non-modifiable lvalues** are lvalues whose values can’t be modified (typically because they are const or constexpr).

An **rvalue** is an expression that is not an l-value. This includes literals (except string literals) and the return values of functions or operators (when returned by value).

A **reference** is an alias for an existing object. Once a reference has been defined, any operation on the reference is applied to the object being referenced. C++ contains two types of references: lvalue references and rvalue references. An **lvalue reference** (commonly just called a **reference**) acts as an alias for an existing lvalue (such as a variable). An **lvalue reference variable** is a variable that acts as a reference to an lvalue (usually another variable).

When a reference is initialized with an object (or function), we say it is **bound** to that object (or function). The object (or function) being referenced is sometimes called the **referent**.

Lvalue references can’t be bound to non-modifiable lvalues or rvalues (otherwise you’d be able to change those values through the reference, which would be a violation of their const-ness). For this reason, lvalue references are occasionally called **lvalue references to non-const** (sometimes shortened to **non-const reference**).

Once initialized, a reference in C++ cannot be **reseated**, meaning it can not be changed to reference another object.

When an object being referenced is destroyed before a reference to it, the reference is left referencing an object that no longer exists. Such a reference is called a **dangling reference**. Accessing a dangling reference leads to undefined behavior.

By using the `const` keyword when declaring an lvalue reference, we tell an lvalue reference to treat the object it is referencing as const. Such a reference is called an **lvalue reference to a const value** (sometimes called a **reference to const** or a **const reference**). Const references can bind to modifiable lvalues, non-modifiable lvalues, and rvalues.

A **temporary object** (also sometimes called an **unnamed object** or **anonymous object**) is an object that is created for temporary use (and then destroyed) within a single expression.

When using **pass by reference**, we declare a function parameter as a reference (or const reference) rather than as a normal variable. When the function is called, each reference parameter is bound to the appropriate argument. Because the reference acts as an alias for the argument, no copy of the argument is made.

The **address-of operator** (&) returns the memory address of its operand. The **dereference operator** (*) returns the value at a given memory address as an lvalue.

A **pointer** is an object that holds a *memory address* (typically of another variable) as its value. This allows us to store the address of some other object to use later. Like normal variables, pointers are not initialized by default. A pointer that has not been initialized is sometimes called a **wild pointer**. A **dangling pointer** is a pointer that is holding the address of an object that is no longer valid (e.g. because it has been destroyed).

Besides a memory address, there is one additional value that a pointer can hold: a null value. A **null value** (often shortened to **null**) is a special value that means something has no value. When a pointer is holding a null value, it means the pointer is not pointing at anything. Such a pointer is called a **null pointer**. The **nullptr** keyword represents a null pointer literal. We can use `nullptr` to explicitly initialize or assign a pointer a null value.

A pointer should either hold the address of a valid object, or be set to `nullptr`. That way we only need to test pointers for null, and can assume any non-null pointer is valid.

A **pointer to a const value** (sometimes called a **pointer to const** for short) is a (non-const) pointer that points to a constant value.

A **const pointer** is a pointer whose address can not be changed after initialization.

A **const pointer to a const value** can not have its address changed, nor can the value it is pointing to be changed through the pointer.

With **pass by address**, instead of providing an object as an argument, the caller provides an object’s address (via a pointer). This pointer (holding the address of the object) is copied into a pointer parameter of the called function (which now also holds the address of the object). The function can then dereference that pointer to access the object whose address was passed.

**Return by reference** returns a reference that is bound to the object being returned, which avoids making a copy of the return value. Using return by reference has one major caveat: the programmer must be sure that the object being referenced outlives the function returning the reference. Otherwise, the reference being returned will be left dangling (referencing an object that has been destroyed), and use of that reference will result in undefined behavior. If a parameter is passed into a function by reference, it’s safe to return that parameter by reference.

If a function returns a reference, and that reference is used to initialize or assign to a non-reference variable, the return value will be copied (as if it had been returned by value).

Type deduction for variables (via the `auto` keyword) will drop any reference or top-level const qualifiers from the deduced type. These can be reapplied as part of the variable declaration if desired.

**Return by address** works almost identically to return by reference, except a pointer to an object is returned instead of a reference to an object.

------

#### Quiz time

**Question #1**

For each of the following expressions on the right side of operator <<, indicate whether the expression is an lvalue or rvalue:

a)

```cpp
std::cout << 5;
```

Literals are rvalues, so `5` is an rvalue

b)

```cpp
int x { 5 };
std::cout << x;
```

The expression `x` identifies variable `x`, so this expression is an lvalue.

c)

```cpp
int x { 5 };
std::cout << x + 1;
```

The expression `x + 1` calculates a temporary value, so this expression is an rvalue.

d)

```cpp
int foo() { return 5; }
std::cout << foo();
```

The return value of a function (when returned by value) is an rvalue.

e)

```cpp
int& max(int &x, int &y) { return x > y ? x : y; }
int x { 5 };
int y { 6 };
std::cout << max(x, y);
```

max() returns an lvalue reference, which is an lvalue.

------

**Question #2**

What is the output of this program?

```cpp
#include <iostream>

int main()
{
	int x{ 5 };
	int y{ 6 };

	int& ref{ x };
	++ref;
	ref = y;
	++ref;

	std::cout << x << ' ' << y;

	return 0;
}
```

```
7 6
```

Remember, references cannot be reseated, so `ref = y` does not make `ref` a reference to `y`. It assigns the value of `y` to the object `ref` is referring to (which is `x`).

------

**Question #3**

Name two reasons why we prefer to pass arguments by const reference instead of by non-const reference whenever possible.

1. A non-const reference parameter can be used to modify the value of the argument. If we do not need this ability, it’s better to pass by const reference so that we don’t accidentally modify the argument.
2. A non-const reference parameter can only accept a modifiable lvalue as an argument. A const reference-parameter can accept a modifiable lvalue, a non-modifiable lvalue, or an rvalue as an argument.

------

**Question #4**

What is the difference between a const pointer and a pointer-to-const?

A const pointer is a pointer whose address can not be changed (it cannot be re-pointed at some other object). However, the value of the object being pointed to can be changed.
A pointer-to-const is a pointer that is pointing at an object whose value can’t be changed through the pointer. However, the pointer can be re-pointed at another object.
