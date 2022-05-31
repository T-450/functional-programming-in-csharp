
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

#### How  functional a language is C#?

- Functions are indeed first-class values in C#;
- C# had support for functions as first-class values from the earliest version of the language through the Delegate type;
- There are some quirks and limitations when it comes to type inference;
- For a long time, this was C#'s greatestshortcoming: having everything mutable by default and no easy way to define immutable types.This all changed with the introduction of records in C# 9 - records allow you to define custom immutable types.

As a result of the features added over time, C# 9 offers good language support for many functional techniques.

#### The functional nature of LINQ
- When C# 3 was released, along with version 3.5 of the .NET Framework, it included a host of features inspired by functional languages, including the LINQ library (System.Linq) and some new language features enabling or enhancing what you could do with LINQ.
- LINQ offers implementations for many common operations on lists (or, more generally, on “sequences,” as instances of IEnumerable should technically be called), the most common of which are mapping, sorting, and filtering;

```cs
// Notice how Where, OrderBy, and Select all take functions
// as arguments and don’t mutate the given IEnumerable but
// return a new IEnumerable instead.

Enumerable.Range(1, 100)
	.Where(i => i % 20 == 0)
	.OrderBy(i => -i)
	.Select(i => $"{i}%")
// => ["100%", "80%", "60%", "40%", "20%"]
```

##### Common operations on sequences:

```cs
// Mapping—Given a sequence and a function, mapping yields a new
// sequence whose elements are obtained by applying the given function
// to each element in the original sequence (in LINQ, this is done with the
// Select method):
	Enumerable.Range(1, 3).Select(i => i * 3) // => [3, 6, 9]

// Filtering—Given a sequence and a predicate, filtering yields a new
// sequence including all the elements from the original sequence that
// satisfy the predicate (in LINQ, this is done with Where):
	Enumerable.Range(1, 10).Where(i => i % 3 == 0) // => [3, 6, 9]
	
// Sorting—Given a sequence and a key-selector function, sorting yields a
// sequence where the elements of the original sequence are ordered by
// the key (in LINQ, this is done with OrderBy and OrderByDescending):
	Enumerable.Range(1, 5).OrderBy(i => -i) // => [5, 4, 3, 2, 1]
```


#### Shorthand syntax for coding functionally

```cs
// C# idioms relevant for FP

// enables unqualified access to the static members of System.Math, like PI and Pow.
using static System.Math;

public record Circle(double Radius)  
{  
    public double Circumference => PI* 2 * Radius; // An expression-bodied property
    public double Area  
    {  
        get  
        {  
            double Square(double d) => Pow(d, 2); // A local function is a method declared within another method.
            return PI * Square(Radius);  
        }  
    }  
}
```

#### IMPORTING STATIC MEMBERS WITH THE USING STATIC DIRECTIVE
- The using static directive introduced in C# 6 allows you
to import the static members of a class;

```cs
using static System.Math;

public double Circumference => PI * 2 * Radius;
```

Why is this important? 

In FP, we prefer functions whose behavior relies only on their input arguments because we can reason about and test these functions in isolation
(contrast this with instance methods, whose implementation typically interacts with instance variables).

#### MORE CONCISE FUNCTIONS WITH EXPRESSION-BODIED MEMBERS
- The expression-bodied syntax was introduced in C# 6 for methods and property getters. It was generalized in C# 7 to also apply to constructors, destructors, getters, and setters:

```cs
// We declare the Circumference property with an expression
// body, introduced with =>, rather than with the usual
// statement body enclosed by curly braces:

public double Circumference => PI * 2 * Radius;
```

- In FP, we tend to write lots of simple functions, many of them one-liners, and then compose these into more complex workflows. Expression-bodied methods allow you to do this with minimal syntactic noise;


#### DECLARING FUNCTIONS WITHIN FUNCTIONS
- Writing lots of simple functions means that many functions
are called from one location only;
- C# allows you to make this explicit by declaring a function within the scope of another function;

```cs
// This code uses a lambda expression to represent the function and assigns it to the square variable.
get
{
	var square = (double d) => Pow(d, 2);
	return PI * square(Radius);
}

// Another possibility is to use local functions, effectively methods declared within a method
get
{
	double Square(double d) => Pow(d, 2);
	return PI * Square(Radius);
}
```

- Both lambda expressions and local functions can refer to variables within the enclosing scope for this reason, the compiler actually generates a class for each local function;
- To mitigate the possible performance impact, declare a local function as static whenever you can:

```cs
static double Square(double d) => Pow(d, 2);
```

#### Language support for tuples
- How are tuples useful in practice, and why are they relevant to FP?
	-  In FP, we tend to break tasks down into small functions;
	- You may end up with a data type whose only purpose is to capture the information returned by one function;
	- It’s impractical to define dedicated types for such structures since don’t correspond to meaningful domain abstractions;

Tuple syntax allows you to elegantly write and consume methods that need to return
more than one value:

```cs
// I’ve defined a method called Partition, which returns a tuple containing both lists
var (even, odd) = nums.Partition(i => i % 2 == 0);
even // => [0, 2, 4, 6, 8]
odd // => [1, 3, 5, 7, 9]
```



#### Pattern matching and record types
- Pattern matching:  
	- Lets you use the switch keyword to match not only on specific values but also on the shape of the data, most importantly its type;

Pattern matching on a value:

```cs
// Notice the clean syntax of a switch expression compared to the traditional switch statement with its clunky use of case, break, and return.

static decimal Vat(Address address, Order order)  
{  
    return Vat(RateByCountry(address.Country), order);  
}  
  
static decimal RateByCountry(string country)  
{  
    return country switch  
    {  
        "it" => 0.22m,  
        "jp" => 0.08m,  
        _ => throw new ArgumentException($"Missing rate for {country}")  
    };  
}  
  
static decimal Vat(decimal rate, Order order)  
{  
    return order.NetPrice * rate;  
}
```

Deconstructing a record in a pattern-matching expression:

```cs

static decimal Vat(Address address, Order order)  
    => address switch  
    {  
	// Address is deconstructed, allowing us to match on the value of its Country.
        Address("de") => DeVat(order),  
        Address(var country) => Vat(RateByCountry(country), order),  
    };
    
static decimal DeVat(Order order)  
    => order.NetPrice * (order.Product.IsFood ? 0.08m : 0.2m);


// It’s possible to simplify the clauses of the switch
static decimal Vat(Address address, Order order)
	=> address switch
		{
			// Because the type of address is known to be Address, you
			// can omit the type.
			("de") _ => DeVat(order),
			(var country) _ => Vat(RateByCountry(country), order),
		};
```


- Records: 
	- Boilerplate-free immutable types with built-in support for creating modified versions;

The following listing shows how you can use record types to
model an Order:

```cs
// Positional records

// A record without a body ends with a semicolon.
record Product(string Name, decimal Price, bool IsFood);

// A record can have a body with additional members.
record Order(Product Product, int Quantity)
{
	public decimal NetPrice => Product.Price * Quantity;
}
```

- Records in C# 9 are reference types, but C# 10 allows you to use record syntax to define value types by simply writing` record struct` rather than just `record`.

- Record structs are mutable, and you have to declare your struct as `readonly record struct` if you want it to be immutable.

- Property patterns:

```cs
// To match on the value of a field by deconstructing the Address; this is called a positional pattern. Now, imagine that your Address type were more complex, including half a dozen fields or so. In this case, a positional pattern would be noisy, as you’d need to include a variable name (at least a discard) for each field. This is where property patterns are better suited. The following code shows how you can match on the value of a property:

static decimal Vat(Address address, Order order)  
    => address switch  
    {  
        { Country: "de" } => DeVat(order),  
        { Country: var c } => Vat(RateByCountry(c), order),  
    };
    
// This syntax offers the advantage that you do not need to change anything if you later add an extra field to Address. In general, property patterns work best with your typical OO entities,
```


- Pattern matching by type

```cs
static decimal Vat(Address address, Order order)  
    => address switch  
    {  
        UsAddress(var state) => Vat(RateByState(state), order),  
        ("de") _ => DeVat(order),  
        (var country) _ => Vat(RateByCountry(country), order),  
    };

static decimal RateByState(string state)  
    => state switch  
    {  
        "ca" => 0.1m,  
        "ma" => 0.0625m,  
        "ny" => 0.085m,  
        _ => throw new ArgumentException($"Missing rate for {state}")  
    };
```





---
### Links:

[Functional Programming in CSharp](https://www.manning.com/books/functional-programming-in-c-sharp-second-edition)
[Out of the Tar Pit by Ben Moseley and Peter Marks](http://mng.bz/xXK7)

---
Source: 
