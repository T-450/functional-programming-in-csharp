### 1.What’s a function, anyway?

- In mathematics, a function is a map between two sets, respectively called the domain and codomain. That is, given an element from its domain, a function yields an element from its codomain.
-  How does this relate to programming functions? In staticallytyped languages like C#, types represent the sets (domain and codomain). For example, if you code the function in figure 2.1, you could use char to represent both the domain and the codomain. The type of your function could then be written as:

```CS  
// The function maps chars to chars or, equivalently, given a char, it yields a char.  
char → char  
```  

- The types for the domain and codomain constitute a function’s interface, also called its type or signature.
- A function signature declares that, given an element from the domain, it will yield an element from the codomain.

### 2.Representing functions in C#

In C# you can represent functions as:
-  Methods (including local functions);
- Delegates;
- Lambda expressions;
- Dictionaries;

##### THE FUNC AND ACTION DELEGATES

.NET includes a couple of delegate families that can represent pretty much any function type - these delegates represent functions of various arities:

```cs  
- Func<R>: Represents a function that takes no arguments and returns a result of type R;  
  
- Func<T1, R>: Represents a function that takes an argument of type T1 and returns a result of type R;  
  
- Func<T1, T2, R>: Represents a function that takes a T1 and a T2 and returns an R;  
```  

Actions are functions that have no return value, such as void methods:

```cs  
Action: Represents an action with no input  
arguments;  
  
Action<T1>: Represents an action with an input  
argument of type T1;  
  
Action<T1, T2> and so on: Represent an action  
with several input arguments;  
```  

The evolution of .NET has been away from custom delegates in favor of the more general Func and Action delegates, take the representation of a predicate:

```cs  
In .NET 2, a Predicate<T> delegate was introduced. For example, the FindAll method used to filter a List<T> expects a Predicate<T  
  
In .NET 3, the Where method, also used for filtering but defined for the more general IEnumerable<T>, doesn’t take a Predicate<T> but simply a Func<T, bool>.  
```  

#####  Delegate vs Func vs Action
```cs
- Using Func is recommended to avoid a proliferation of delegate types that represent the same function signature; 

- There’s still something to be said in favor of the expressiveness of custom delegates;  

- Predicate<T>  conveys intent more clearly than Func<T, bool> and is closer to the spoken language;
```

##### Function Arity
Arity is a funny word that refers to the **number of arguments that a function accepts**:

	- A nullary function takes no arguments.
	- A unary function takes one argument.
	- A binary function takes two arguments.
	- A ternary function takes three arguments.

#### LAMBDA EXPRESSIONS

-  Lambda expressions, called lambdas for short, are used to declare a function inline;

- If your function is short, and you don’t need to reuse it elsewhere, lambdas offer the most attractive notation;

```cs
// Declaring a function inline with a lambda
var list = Enumerable.Range(1, 10).Select(i => i * 3).ToList();

list // => [3, 6, 9, 12, 15, 18, 21, 24, 27, 30]

list.Sort((l, r) => l.ToString().CompareTo(r.ToString()));

list // => [12, 15, 18, 21, 24, 27, 3, 30, 6, 9]
```

- Delegates and lambdas have access to the variables in the scope in which they’re declared - useful when leveraging closures;

```cs
// A lambda accessing a variable from the enclosing scope
var days = Enum.GetValues(typeof(DayOfWeek)).Cast<DayOfWeek>();

// => [Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday]
IEnumerable<DayOfWeek> daysStartingWith(string s)
	=> days.Where(d  => d.ToString().StartsWith(s));
	 
daysStartingWith("S") // => [Sunday, Saturday]
```

#### Anonymous methods

Anonymous methods. These allow you to create a delegate like this:

```cs
Comparison<int> alphabetically = delegate (int l, int r)
	{
		return l.ToString().CompareTo(r.ToString());
	};
```

- Anonymous methods were superseded in C# 3 when lambda expressions provided a more concise syntax for doing the same. Anonymous methods survive as a vestigial feature of the language, but their use is discouraged.

#### DICTIONARIES
- Dictionaries are fittingly also called maps (or hash tables). They’re data structures that provide a direct representation of a function.
- Dictionaries are appropriate for representing functions that are completely arbitrary, where the mappings can’t be computed but must be stored exhaustively.

The following listing shows how you could map Boolean values to their names in French:

```cs
// A dictionary exhaustively representing a function
var frenchFor = new Dictionary<bool, string>
{
	[true] = "Vrai",
	[false] = "Faux",
};
// Function application is performed with a lookup.
frenchFor[true] // => "Vrai"
```

- Functions can be represented with dictionaries also makes it possible to optimize computationally expensive functions by storing their computed results in a dictionary instead of recomputing them every time. This technique iscalled **memoization**.


### 3. Higher-order functions (HOFs)

- HOFs are functions that take other functions as inputs or
  return a function as output or both.
- HOFs can help with the separation of concerns in cases where logic can’t otherwise be easily separated.

```cs
// Where, a HOF that iteratively applies the given predicate
public static IEnumerable<T> Where<T>( this IEnumerable<T> ts, Func<T, bool> predicate)  
{  
    foreach (T t in ts)  
    {  
        if (predicate(t)) yield return t;  
    }   
}
```

- Optional execution is another good candidate for HOFs. For example, imagine a method that looks up an element from the cache. A delegate can be provided and can be invoked in case of a cache miss. The following listing shows how this is done:

```cs
// A HOF that optionally invokes the given function
class Cache<T> where T : class
{
	public T Get(Guid id) => //...
	public T Get(Guid id, Func<T> onMiss) => Get(id) ?? onMiss();
}
// The logic in onMiss could involve an expensive operation, such as a database call.
```

#### Adapter functions

- Some HOFs don’t apply the given function at all, but rather return a new function that’s somehow related to the function given as an argument.  For example, say you have a function that performs integer division:

```cs
var divide = (int x, int y) => x / y;

divide(10, 2) // => 5
```

- You can write a generic HOF that modifies any binary function by swapping the order of its arguments:

```cs
static Func<T2, T1, R> SwapArgs<T1, T2, R>(this Func<T1, T2, R> f)
	=> (t2, t1) => f(t1, t2);
```

- You can now modify the original division function by applying SwapArgs:

```cs
var divideBy = divide.SwapArgs();

divideBy(2, 10) // => 5
```

If you don’t like the interface of a function, you can call it via another function that provides an interface that better suits your needs - or adapter functions.

####  Functions that create other functions

Sometimes you’ll write functions whose primary purpose is to create other functions: You can think of them as function factories;

```cs
// The following example uses a lambda to filter a sequence of numbers, keeping only those divisible by 2:
var range = Enumerable.Range(1, 20);

range.Where(i => i % 2 == 0); // => [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

- What if you wanted something more general, like being able
  to filter for numbers divisible by any number n?
- You could define a function that takes n and yields a suitable predicate that evaluates whether any given number is divisible by n:

```cs
using static System.Linq.Enumerable;

Func<int, bool> isMod(int n) => i => i % n == 0;

Range(1, 20).Where(isMod(2)) // => [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
Range(1, 20).Where(isMod(3)) // => [3, 6, 9, 12, 15, 18]

// Notice how you’ve gained not only in generality but also in readability!
```

#### Using HOFs to avoid duplication
- Another common use case for HOFs is to encapsulate setup and teardown operations.

```cs
// Connecting to a DB with some setup and teardown
string connString = "myDatabase";

var conn = new SqlConnection(connString));
conn.Open();

// interact with the database...
conn.Close();
conn.Dispose();
```

- The setup and teardown are always identical, regardless of whether you’re reading or writing to the database, or performing one or many actions. The preceding code is usually written with a using block like this:

```cs
// This is both shorter and better, but it’s still essentially the same
using (var conn = new SqlConnection(connString))
{
	conn.Open(); // interact with the database...
}
```

- You’re looking to write a function that performs setup and teardown and that’s parameterized on what to do in between.

```cs
// Encapsulating setup and teardown of the DB connection into a HOF
public static class ConnectionHelper {  
    public static R Connect<R> (string connString, Func<IDbConnection, R> f)  
    {  
        using SqlConnection conn = new (connString);  
        conn.Open();  
        return f(conn);  
    }  
}

// Usage
using Dapper;  
using static ConnectionHelper;  
  
public class DbLogger  
{  
    string connString;  
  
    public void Log(LogMessage message)   
=> Connect(connString, c => c.Execute("sp_create_log", message, commandType: CommandType.StoredProcedure));  
  
    string sql = @"SELECT * FROM [Logs] WHERE [Timestamp] > @since";  
  
    public IEnumerable<LogMessage> GetLogs(DateTime since)  
        => Connect(connString, c => c.Query<LogMessage>(sql, new {since = since}));  
}
```


FP leverages higher-order functions (HOFs), which are functions that take other functions as input or output); hence, the necessity for the language to have functions as first-class values.
