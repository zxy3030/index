## **The Standard Template Library**

### 21.1 — The Standard Library

Congratulations! You made it all the way through the primary portion of the tutorial! In the preceding lessons, we covered many of the principal C++ language features (including a few from the C++11/14/17 extension to the language).

So the obvious question is, “what next?”. One thing you’ve probably noticed is that an awful lot of programs use the same concepts over and over again: loops, strings, arrays, sorting, etc… You’ve probably also noticed that programs written using non-class versions of containers and common algorithms are error-prone. The good news is that C++ comes with a library that is chock full of reusable classes for you to build programs out of. This library is called The C++ Standard Library.

**The Standard Library**

The Standard library contains a collection of classes that provide templated containers, algorithms, and iterators. If you need a common class or algorithm, odds are the standard library has it. The upside is that you can take advantage of these classes without having to write and debug the classes yourself, and the standard library does a good job providing reasonably efficient versions of these classes. The downside is that the standard library is complex, and can be a little intimidating since everything is templated.

Fortunately, you can bite off the standard library in tiny pieces, using only what you need from it, and ignore the rest until you’re ready to tackle it.

In the next few lessons, we’ll take a high-level look at the types of containers, algorithms, and iterators that the standard library provides. Then in subsequent lessons, we’ll dig into some of the specific classes.

------

### 21.2 — STL containers overview

By far the most commonly used functionality of the STL library are the STL container classes. If you need a quick refresher on container classes, check out lesson [16.6 -- Container classes](https://www.learncpp.com/cpp-tutorial/container-classes/).

The STL contains many different container classes that can be used in different situations. Generally speaking, the container classes fall into three basic categories: Sequence containers, Associative containers, and Container adapters. We’ll just do a quick overview of the containers here.

#### **Sequence Containers**

Sequence containers are container classes that maintain the ordering of elements in the container. A defining characteristic of sequence containers is that you can choose where to insert your element by position. The most common example of a sequence container is the array: if you insert four elements into an array, the elements will be in the exact order you inserted them.

As of C++11, the STL contains 6 sequence containers: std::vector, std::deque, std::array, std::list, std::forward_list, and std::basic_string.

- If you’ve ever taken physics, you probably are thinking of a vector as an entity with both magnitude and direction. The unfortunately named **vector** class in the STL is a dynamic array capable of growing as needed to contain its elements. The vector class allows random access to its elements via operator[], and inserting and removing elements from the end of the vector is generally fast.

  The following program inserts 6 numbers into a vector and uses the overloaded [] operator to access them in order to print them.

  ```cpp
  #include <vector>
  #include <iostream>
  
  int main()
  {
  
      std::vector<int> vect;
      for (int count=0; count < 6; ++count)
          vect.push_back(10 - count); // insert at end of array
  
      for (int index=0; index < vect.size(); ++index)
          std::cout << vect[index] << ' ';
  
      std::cout << '\n';
  }
  ```

  This program produces the result:
  10 9 8 7 6 5

- The **deque** class (pronounced “deck”) is a double-ended queue class, implemented as a dynamic array that can grow from both ends.

  ```cpp
  #include <iostream>
  #include <deque>
  
  int main()
  {
      std::deque<int> deq;
      for (int count=0; count < 3; ++count)
      {
          deq.push_back(count); // insert at end of array
          deq.push_front(10 - count); // insert at front of array
      }
  
      for (int index=0; index < deq.size(); ++index)
          std::cout << deq[index] << ' ';
  
      std::cout << '\n';
  }
  ```

  This program produces the result:

  8 9 10 0 1 2

- A **list** is a special type of sequence container called a doubly linked list where each element in the container contains pointers that point at the next and previous elements in the list. Lists only provide access to the start and end of the list -- there is no random access provided. If you want to find a value in the middle, you have to start at one end and “walk the list” until you reach the element you want to find. The advantage of lists is that inserting elements into a list is very fast if you already know where you want to insert them. Generally iterators are used to walk through the list.

  We’ll talk more about both linked lists and iterators in future lessons.

- Although the STL **string** (and wstring) class aren’t generally included as a type of sequence container, they essentially are, as they can be thought of as a vector with data elements of type char (or wchar).

------

#### **Associative Containers**

Associative containers are containers that automatically sort their inputs when those inputs are inserted into the container. By default, associative containers compare elements using operator<.

- A **set** is a container that stores unique elements, with duplicate elements disallowed. The elements are sorted according to their values.
- A **multiset** is a set where duplicate elements are allowed.
- A **map** (also called an associative array) is a set where each element is a pair, called a key/value pair. The key is used for sorting and indexing the data, and must be unique. The value is the actual data.
- A **multimap** (also called a dictionary) is a map that allows duplicate keys. Real-life dictionaries are multimaps: the key is the word, and the value is the meaning of the word. All the keys are sorted in ascending order, and you can look up the value by key. Some words can have multiple meanings, which is why the dictionary is a multimap rather than a map.

------

#### **Container Adapters**

Container adapters are special predefined containers that are adapted to specific uses. The interesting part about container adapters is that you can choose which sequence container you want them to use.

- A **stack** is a container where elements operate in a LIFO (Last In, First Out) context, where elements are inserted (pushed) and removed (popped) from the end of the container. Stacks default to using deque as their default sequence container (which seems odd, since vector seems like a more natural fit), but can use vector or list as well.
- A **queue** is a container where elements operate in a FIFO (First In, First Out) context, where elements are inserted (pushed) to the back of the container and removed (popped) from the front. Queues default to using deque, but can also use list.
- A **priority queue** is a type of queue where the elements are kept sorted (via operator<). When elements are pushed, the element is sorted in the queue. Removing an element from the front returns the highest priority item in the priority queue.

------

### 21.3 — STL iterators overview

An **Iterator** is an object that can traverse (iterate over) a container class without the user having to know how the container is implemented. With many classes (particularly lists and the associative classes), iterators are the primary way elements of these classes are accessed.

An iterator is best visualized as a pointer to a given element in the container, with a set of overloaded operators to provide a set of well-defined functions:

- **Operator\*** -- Dereferencing the iterator returns the element that the iterator is currently pointing at.
- **Operator++** -- Moves the iterator to the next element in the container. Most iterators also provide `Operator--` to move to the previous element.
- **Operator== and Operator!=** -- Basic comparison operators to determine if two iterators point to the same element. To compare the values that two iterators are pointing at, dereference the iterators first, and then use a comparison operator.
- **Operator=** -- Assign the iterator to a new position (typically the start or end of the container’s elements). To assign the value of the element the iterator is pointing at, dereference the iterator first, then use the assign operator.

Each container includes four basic member functions for use with Operator=:

- **begin()** returns an iterator representing the beginning of the elements in the container.
- **end()** returns an iterator representing the element just past the end of the elements.
- **cbegin()** returns a const (read-only) iterator representing the beginning of the elements in the container.
- **cend()** returns a const (read-only) iterator representing the element just past the end of the elements.

It might seem weird that end() doesn’t point to the last element in the list, but this is done primarily to make looping easy: iterating over the elements can continue until the iterator reaches end(), and then you know you’re done.

Finally, all containers provide (at least) two types of iterators:

- **container::iterator** provides a read/write iterator
- **container::const_iterator** provides a read-only iterator

Lets take a look at some examples of using iterators.

------

#### **Iterating through a vector**

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> vect;
    for (int count=0; count < 6; ++count)
        vect.push_back(count);

    std::vector<int>::const_iterator it; // declare a read-only iterator
    it = vect.cbegin(); // assign it to the start of the vector
    while (it != vect.cend()) // while it hasn't reach the end
    {
        std::cout << *it << ' '; // print the value of the element it points to
        ++it; // and iterate to the next element
    }

    std::cout << '\n';
}
```

This prints the following:

```
0 1 2 3 4 5
```

------

#### **Iterating through a list**

Now let’s do the same thing with a list:

```cpp
#include <iostream>
#include <list>

int main()
{

    std::list<int> li;
    for (int count=0; count < 6; ++count)
        li.push_back(count);

    std::list<int>::const_iterator it; // declare an iterator
    it = li.cbegin(); // assign it to the start of the list
    while (it != li.cend()) // while it hasn't reach the end
    {
        std::cout << *it << ' '; // print the value of the element it points to
        ++it; // and iterate to the next element
    }

    std::cout << '\n';
}
```

This prints:

```
0 1 2 3 4 5
```

Note the code is almost identical to the vector case, even though vectors and lists have almost completely different internal implementations!

------

#### **Iterating through a set**

In the following example, we’re going to create a set from 6 numbers and use an iterator to print the values in the set:

```cpp
#include <iostream>
#include <set>

int main()
{
    std::set<int> myset;
    myset.insert(7);
    myset.insert(2);
    myset.insert(-6);
    myset.insert(8);
    myset.insert(1);
    myset.insert(-4);

    std::set<int>::const_iterator it; // declare an iterator
    it = myset.cbegin(); // assign it to the start of the set
    while (it != myset.cend()) // while it hasn't reach the end
    {
        std::cout << *it << ' '; // print the value of the element it points to
        ++it; // and iterate to the next element
    }

    std::cout << '\n';
}
```

This program produces the following result:

```
-6 -4 1 2 7 8
```

Note that although populating the set differs from the way we populate the vector and list, the code used to iterate through the elements of the set was essentially identical.

------

#### **Iterating through a map**

This one is a little trickier. Maps and multimaps take pairs of elements (defined as a std::pair). We use the make_pair() helper function to easily create pairs. std::pair allows access to the elements of the pair via the first and second members. In our map, we use first as the key, and second as the value.

```cpp
#include <iostream>
#include <map>
#include <string>

int main()
{
	std::map<int, std::string> mymap;
	mymap.insert(std::make_pair(4, "apple"));
	mymap.insert(std::make_pair(2, "orange"));
	mymap.insert(std::make_pair(1, "banana"));
	mymap.insert(std::make_pair(3, "grapes"));
	mymap.insert(std::make_pair(6, "mango"));
	mymap.insert(std::make_pair(5, "peach"));

	auto it{ mymap.cbegin() }; // declare a const iterator and assign to start of vector
	while (it != mymap.cend()) // while it hasn't reach the end
	{
		std::cout << it->first << '=' << it->second << ' '; // print the value of the element it points to
		++it; // and iterate to the next element
	}

	std::cout << '\n';
}
```

This program produces the result:

```
1=banana 2=orange 3=grapes 4=apple 5=peach 6=mango
```

Notice here how easy iterators make it to step through each of the elements of the container. You don’t have to care at all how map stores its data!

------

#### **Conclusion**

Iterators provide an easy way to step through the elements of a container class without having to understand how the container class is implemented. When combined with STL’s algorithms and the member functions of the container classes, iterators become even more powerful. In the next lesson, you’ll see an example of using an iterator to insert elements into a list (which doesn’t provide an overloaded operator[] to access its elements directly).

One point worth noting: Iterators must be implemented on a per-class basis, because the iterator does need to know how a class is implemented. Thus iterators are always tied to specific container classes.

------

### 21.4 — STL algorithms overview

In addition to container classes and iterators, STL also provides a number of generic algorithms for working with the elements of the container classes. These allow you to do things like search, sort, insert, reorder, remove, and copy elements of the container class.

Note that algorithms are implemented as functions that operate using iterators. This means that each algorithm only needs to be implemented once, and it will generally automatically work for all containers that provides a set of iterators (including your custom container classes). While this is very powerful and can lead to the ability to write complex code very quickly, it’s also got a dark side: some combination of algorithms and container types may not work, may cause infinite loops, or may work but be extremely poor performing. So use these at your risk.

STL provides quite a few algorithms -- we will only touch on some of the more common and easy to use ones here. The rest (and the full details) will be saved for a chapter on STL algorithms.

To use any of the STL algorithms, simply include the algorithm header file.

------

#### **min_element and max_element**

The `std::min_element` and `std::max_element` algorithms find the min and max element in a container class. `std::iota` generates a contiguous series of values.

```cpp
#include <algorithm> // std::min_element and std::max_element
#include <iostream>
#include <list>
#include <numeric> // std::iota

int main()
{
    std::list<int> li(6);
    // Fill li with numbers starting at 0.
    std::iota(li.begin(), li.end(), 0);

    std::cout << *std::min_element(li.begin(), li.end()) << ' '
              << *std::max_element(li.begin(), li.end()) << '\n';

    return 0;
}
```

Prints:

0 5

------

#### **find (and list::insert)**

In this example, we’ll use the `std::find()` algorithm to find a value in the list class, and then use the list::insert() function to add a new value into the list at that point.

```cpp
#include <algorithm>
#include <iostream>
#include <list>
#include <numeric>

int main()
{
    std::list<int> li(6);
    std::iota(li.begin(), li.end(), 0);

    // Find the value 3 in the list
    auto it{ std::find(li.begin(), li.end(), 3) };

    // Insert 8 right before 3.
    li.insert(it, 8);

    for (int i : li) // for loop with iterators
        std::cout << i << ' ';

    std::cout << '\n';

    return 0;
}
```

This prints the values

```
0 1 2 8 3 4 5
```

When a searching algorithm doesn’t find what it was looking for, it returns the end iterator.
If we didn’t know for sure that 3 is an element of `li`, we’d have to check if `std::find` found it before we use the returned iterator for anything else.

```cpp
if (it == li.end())
{
  std::cout << "3 was not found\n";
}
else
{
  // ...
}
```

------

#### **sort and reverse**

In this example, we’ll sort a vector and then reverse it.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main()
{
    std::vector<int> vect{ 7, -3, 6, 2, -5, 0, 4 };

    // sort the vector
    std::sort(vect.begin(), vect.end());

    for (int i : vect)
    {
        std::cout << i << ' ';
    }

    std::cout << '\n';

    // reverse the vector
    std::reverse(vect.begin(), vect.end());

    for (int i : vect)
    {
        std::cout << i << ' ';
    }

    std::cout << '\n';

    return 0;
}
```

This produces the result:

```
-5 -3 0 2 4 6 7
7 6 4 2 0 -3 -5
```

Alternatively, we could pass a custom comparison function as the third argument to `std::sort`. There are several comparison functions in the <functional> header which we can use so we don’t have to write our own. We can pass `std::greater` to `std::sort` and remove the call to `std::reverse`. The vector will be sorted from high to low right away.

Note that `std::sort()` doesn’t work on list container classes -- the list class provides its own `sort()` member function, which is much more efficient than the generic version would be.

------

#### **Conclusion**

Although this is just a taste of the algorithms that STL provides, it should suffice to show how easy these are to use in conjunction with iterators and the basic container classes. There are enough other algorithms to fill up a whole chapter!