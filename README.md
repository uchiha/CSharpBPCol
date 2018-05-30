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