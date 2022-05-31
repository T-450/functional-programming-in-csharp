
#### Notes based on the book Functional Programming in C# by Enrico Buonanno

###  What is this thing called functional programming?

- At a high level, it's a programming style that emphasizes functions while avoiding state mutation.
- FP includes two fundamental concepts:
	1. Functions as first-class value;
	2. Avoiding state mutation;

### Functions as first-class values
In a language where functions are first-class values, you can:
-  Use functions as inputs or outputs of other functions;
-  Can assign them to variables;
-  Can store them in collections;
-  You can do with functions all the operations that you can do with values of any other
type;

Consider this simple example of using a function as a first-class value:

```cs
var triple = (int x) => x * 3; // Defines a function that returns the triple of a given integer

var range = Enumerable.Range(1, 3); // Creates a list with the values [1, 2, 3]

var triples = range.Select(triple); //applies triple to all the values in range

triples // => [3, 6, 9]
```

- The example demonstrates that functions are indeed first-class values in C# because you can assign the multiply-by-3 function to the variable triple and give it as an argument to `Select`;
- Treating functions as values allows you to write powerful and concise code;

###  Avoiding state mutation

- The term mutation indicates that a value is changed in place, updating a value stored somewhere in memory.

```cs
// The following code creates and populates an array, 
// and then it updates one of the array’s values in place:

int[] nums = { 1, 2, 3 };

nums[0] = 7;

nums // => [7, 2, 3
```

- Such updates are also called **destructive updates** because the value stored prior to the update is destroyed;
- Following this principle, sorting or filtering a list should not modify the list in place but should create a new, suitably filtered or sorted list without affecting the original;

```cs
// Functional approach: Where and OrderBy create new lists
var isOdd = (int x) => x % 2 == 1;

int[] original = { 7, 6, 1 };

var sorted = original.OrderBy(x => x);

var filtered = original.Where(isOdd);

original // => [7, 6, 1] The original list isn’t affected
sorted // => [1, 6, 7] Sorting and filtering yielded new lists.
filtered // => [7, 1] Sorting and filtering yielded new lists.


// Nonfunctional approach: List<T>.Sort sorts the list in place
int[] original = { 5, 7, 1 };

Array.Sort(original); // The original ordering is destroyed

original // => [1, 5, 7]
```


###  Writing programs with strong guarantees

The below example demonstrate why avoiding state mutation is also hugely beneficial—it eliminates many complexities caused by mutable state:

```cs
// Mutating state from concurrent processes

using static System.Linq.Enumerable; 
// lets you call Range and WriteLine without full qualification
using static System.Console;

var nums = Range(-10000, 20001).Reverse().ToArray();
// => [10000, 9999, ... , -9999, -10000]

var task1 = () => WriteLine(nums.Sum());

var task2 = () => { Array.Sort(nums); WriteLine(nums.Sum()); };

Parallel.Invoke(task1, task2); //Executes both tasks in paralle
// prints: 5004 (or another unpredictable value)
// 0
```

In the example above you define nums to be an array of all integers between 10,000 and -10,000; their sum should obviously be 0. You then create two tasks:
- task1 computes and prints the sum.
- task2 first sorts the array and then computes and prints the sum.

- Each of these tasks correctly computes the sum if run independently. When you run both tasks in parallel, however, task1 comes up with an incorrect and unpredictable result. It’s easy to see why. As task1 reads the numbers in the array to compute the sum, task2 is reordering the elements in the array. That’s somewhat like trying to read a book while somebody else flips the pages: you’d be reading some well-mangled sentences!

The example below use LINQ’s OrderBy method, instead of sorting the list in place:

```cs
// Functional approach

// Modifying data in place can give concurrent threads an incorrect view of the data
var task3 = () => WriteLine(nums.OrderBy(x => x).Sum());

Parallel.Invoke(task1, task3);
// prints: 0
// 0

```

- As you can see, using LINQ’s functional implementation gives you a predictable result, even when you execute the tasks in parallel;
- Because task3 isn’t modifying the original array but rather creating a completely new view of the data, which is sorted: task1 and task3 read from the original array concurrently, but concurrent reads don’t cause any inconsistencies;


#### How FP differs from OOP in terms of structuring a large, complex application.

The difficult art of structuring a complex application relies on the following principles:
- **Modularity**—Software should be composed of discrete, reusable components;
- **Separation of concerns** — Each component should only do one thing;
- **Layering** —High-level components can depend on low-level components but not vice versa;
- **Loose coupling** —A component shouldn’t know about the internal details of the components it depends on; therefore, changes to a component shouldn’t affect components that depend on it;

These principles are also in no way specific to OOP, so the same principles can be used to structure an application written in the functional style. The difference will be in what the components are and which APIs they expose.





---
### Links:

[Functional Programming in CSharp](https://www.manning.com/books/functional-programming-in-c-sharp-second-edition)
[Out of the Tar Pit by Ben Moseley and Peter Marks](http://mng.bz/xXK7)

---
Source: 
