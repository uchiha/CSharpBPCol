On best practices  :/
Array Initialization
Do:
	- Use collection initializers e.g. string[] colors = {"yellow", "blue", "white"}
Dont:
	- Manually populate an array

foreach sample:
string[] colorOptions = { "Red", "Espresso", "White", "Navy" };

foreach (var color in colorOptions)
{
    Console.WriteLine($"The color is {color}");
}

==> foreach can easily iterate through all array elements
Limitations:
- iterates through ALL the elements, we can't just iterate through some. Though logic like continue can be used to skip some entries, BUT it will still iterate through all of them.
- also, with foreach, the element itself can't be changed, in our example "color" can't be changed. 
foreach (var color in colorOptions)
{
    color = color.ToLower(); // this will be an error in C#, because this is a foreach iteration variable!
    Console.WriteLine($"The color is {color}");
}
- HOWEVER, with the plain for loop, we can change the value per iteration:
for (int i = 0; i < colorOptions.Length; i++)
{
    colorOptions[i] = colorOptions[i].ToLower();
}
  we can also change this code to process some of the elements only. Using for provides much more flexibility when working with arrays.

BOTTOMLINE:
- use foreach to quickly iterate over all array elements.
- use for to iterate a subset of the elements of the array or modify array elements as they're iterated.
- in C#, all arrays derive from System.Array class

/***** GENERICS *****/
Generics are:
	- writing code without specifying data types
	- yet "TYPE-SAFE"
	- a way to make our code generic

WHY have generics!
	- DRY! -> Don't Repeat Yourself
	
[30052018]
The case for generics in this codebase looked like...
- in the Acme solution, in the Common project, one of the classes is OperationResult. Its purpose is to provide an object we can use to return multiple values from a method. Initially, it provides a boolean success flag and an associated message string. 
- in the Biz project, specifically Vendor class, we have a method (PlaceOrder) that returns an instance of the OperationResult, which will allow us to return both success flag and the text of the order. This way, we can display the order text to the user as part of the order confirmation.
- On the Product.cs class, the method CalculateSuggestedPrice, with guard clauses. The thing is, how can we return both the message and the value from this object? We can use OperationResult class, however, its returning a bool and a string. We need to return a decimal and a string.
- That is why on the same file, we create a 2nd class OperationResultDecimal, VERY SIMILAR to OperationResult, but just returns decimal and string. This defines a decimal result and a message. 
- so that is why, instead of CalculatedSuggestedPrice return decimal, we can say it will return OperationResultDecimal type of an object. As in:
  ----> var operationalRes = new OperationResultDecimal(value, message);
        return operationalRes;
- the problem now? We have to OperationalResult classes. One that returns a bool and string and another that returns decima and string. But what if we need another return type? Will we create a new very similar OperationalResult class?
- this breaks the [D]on't [R]epeat [Y]ourself principle.
- it would be nice if the datatype that was needed to be returned would have been a variable. And we can use OperationResult class to return any of our desired type??
- that path would lead to a lot of duplicated code...
- the solution is to use Generics, to build a single reusable OperationResult class that will work on any datatype.
- first, instead of Success, rename to Result (the Success property of OperationResult class) so its more applicable in any data type.
  2nd, change the return type of Result from boolean to T. T is a convention to represent "that" type variable. This is called the generic type. By using this we can reuse this property to return a string, bool or whatever. This will be an error, because we are using "T" but havent defined it. We need to define this.
  3rd define it in the class signature as in...
  -----> public class OperationResult<t>
  <T> is called type parameter and syntax is read as "public class OperationResult of T". THis is now a generic class.
  4th, the argument type in the 2nd constructor should be changed also from bool to T:
  -----> public OperationResult(T result, string message) : this()
  NOW ITS A COMPLETE GENERIC CLASS!
- at this point, there was a lot of changes in the classes in the test project etc to use generic class..
- we, at this point, created a generic class and property ONLY!

[31052018]
- Generic methods can also be defined.
- basically:
  <DO>
  -> use generics to build  reusable type-neutral methods.
  -> Use T as the type parameter for methods with one type parameter.
  -> Prefix descriptive type parameter names with T e.g. TReturn or TParameter
  -> Define the type parameter(s) on the method signature. 
  <DON'T>
  -> use generics when not needed (when you are certain that the method will never be used with other data types). Using them unnecessarily will cause confusion and hard to find bugs.
  -> use single letter type parameter names when there are multiple types to be declared.
- example is the RetrieveValue method in VendorRepository.cs class file. It has has a return type of "T" and one of the parameters has a type of "T" as well:
  ----->
  public T RetrieveValue<T>(string sql, T defaultValue)
  {
     // call the db to retrieve the value
     // if no value returned, return the default value
      T value = defaultValue;

      return value;
  }
- Generic constraints allow us to limit the types that can be used with our generic types.
  The following are the generic constraints
  # Generic Constraints #								# Constraints T to #
   where T : struct										 Value type
   where T : class										 Reference Types
   where T : new()										 Type with parameterless constructor
   where T : Vendor										 Be or derive from Vendor (Vendor should be instantiable and can be inherited from)
   where T : IVendor									 Be or implement the IVendor interface
- an example of generic constraint in a class
  -----> public class OperationResult<T> where T: struct
  the generic constraint is applied by adding a where clause at the end of the signature. THe value after the where keyword is the generic type parameter name, then a colon then the constraint. THe struct keyword limits the generic type arguments to value types such as in or bool.
- if a generic type parameter has multiple constraints, separate them with commas
  -----> public T Populate<T>(string sql) where T : class, new()
- if there are multiple generic parameters with constraints, add multiple where clauses
  -----> public T RetrieveValue<T, V>(string sql, V parameter)where T: struct where V: struct
  
/* Generic Lists */
- Bottom line: Generic lists is the BEST CHOICE for managing an in-memory collection of elements! Its flexibility makes it easy to work with! ONLY use an array if you need MULTIPLE DIMENSION or to squeeze out a bit more performance WHEN WORKING WITH A LARGE FIXED NUMBER OF ELEMENTS.
- List best practice:
  <DO>
  -> use generic lists to manage collection
  -> use Add over Insert if possible
  -> use plural variable name for the lists
  <DON'T>
  -> if possible remove elements. Remove is slow. It first does a sequential search for the element before it can remove it. If you need many inserts and removal, consider a different type of list, such as a linked list.
- just use list initializer syntax:
  -----> var colorOptions = new List<string>() {"Red", "Espresso", "White", "Navy"};
- in this new commit, in the initialize a list of objects, we have the following changes
  VendorRepository.cs - we've set a variable that will hold a list of Vendor objects (generic)
                      - we've created a method that will give the user a list of instantiated list of Vendor Objects.
  VendorRepositoryTests.cs - a simple test that will check the count.