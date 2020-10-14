# **C#/.NET Programming Standards**

## **Introduction**

This guide has been written to provide AMU programmers with a set of coding standards and conventions to follow when developing new .NET applications. These guidelines borrow from Microsoft's  [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions).

These .NET programming standards and best practices promote a higher level of maintainability and readability within the software.

AMU benefits:

- Create a consistent look to the code, so that readers can focus on content, not layout.
- Create a common reference point for development and technical discussions.
- Reduce learning curve for new developers.
- Demonstrate C# best practices and reduce common coding issues.



## Layout Conventions

Microsoft examples and samples conform to the following conventions:

- Use the default Code Editor settings (smart indenting, four-character indents, tabs saved as spaces). For more information, see [Options, Text Editor, C#, Formatting](https://docs.microsoft.com/en-us/visualstudio/ide/reference/options-text-editor-csharp-formatting).  Note: this is a departure from a two-space rule AMU used in the past, and only applies to C#.
- Write only one statement per line.
- Write only one declaration per line.
- Vertically align curly brackets.
- If continuation lines are not indented automatically, indent them one tab stop (four spaces).
- Add at least one blank line between method definitions and property definitions.
- Brackets are major structural constructs and should always go on their own line.
- A semicolon terminates a statement, and goes on the same line as the end of the statement.
- Declare member variables at the top of a class, with static variables at the very top.

**<u>Always Use Access Modifiers</u>**

Use the lowest necessary modifier - principle of least privilege.

**<u>Use Auto Properties</u>**

Unless you need control over the backing field, don't create one. Use the auto property.

**<u>Code File Organization</u>**

Use a single file for each class, interface, struct or enum.



## Naming Conventions

**<u>Pascal Casing</u>**

The first letter in the identifier and the first letter of each subsequent concatenated word are capitalized. Use Pascal casing for class names, file names, namespace names, method names and public member names.

`FreezingPoint	`

**<u>Camel Casing</u>**

The first letter in the identifier is lowercase and the first letter of each subsequent concatenated word is capitalized. Use Camel casing for local variables and method parameters that are not publicly available.

`freezingPoint`

**<u>Snake Casing</u>**

The first letter of each word is lowercase. Each space in the identifier is replaced by an underscore (_) character. This convention may be used on descriptive names for unit test classes.

`snake_case_used_for_test_classes`

**<u>Upper Snake Casing</u>**

All letters in the identifier are capitalized (aka upper snake case). This is only used for constants. For example:

`FREEZING_POINT`

**<u>Do's and Don'ts</u>** 

- Do not use Hungarian notation to add type identification to identifiers (e.g., string strName).
- Do not use underscores in identifiers. *Exceptions: unit test class names, and private static variables* *(e.g., _injectedService).*
- Do prefix interface names with the capital letter I (e.g., IServiceName).
- Do name source files according to their main classes.
- Do use singular names for enums. *Exception: bit field enums.*



## Commenting Conventions

- Place comments on a separate line, not at the end of a line of code.

- Begin comment text with an uppercase letter.

- End comment text with a period.

- Insert one space between the comment delimiter (//) and the comment text, as shown in the following example.

  ```csharp
  // The fox jumped over the dog.
  ```

- Do not create formatted blocks of asterisks around comments.

  

## Language Guidelines

**<u>String Data Types</u>**

- Use [string interpolation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated) to concatenate short strings.

- To append strings in loops, especially with large amounts of text, use a StringBuilder object.

  

<u>**Implicitly Typed Local Variables**</u>

Local variables can be declared without giving an explicit type. The var

- Use implicit typing (i.e., var) when the type of the variable is obvious from the right side of the assignment, or when precise type is not important.

   ```csharp
   // When the type of a variable is clear from the context, use var
   // in the declaration.
   var var1 = "This is clearly a string.";
   var var2 = 27;
   ```




- Do not use var when the type is not apparent from the right side of the assignment.

  ```csharp
// When the type of a variable is not clear from the context, use an
// explicit type. You generally don't assume the type clear from a method name.
// A variable type is considered clear if it's a new operator or an explicit cast.
int var3 = Convert.ToInt32(Console.ReadLine());
int var4 = ExampleClass.ResultSoFar();
var myList = new List<string>();
  ```




- Do not rely on the variable name to specify the type of the variable. It might not be correct.

  ```csharp
  // Naming the following variable inputInt is misleading.
  // It is a string.
  var inputInt = Console.ReadLine();
  Console.WriteLine(inputInt);
  ```

  

- Use implicit typing to determine the type of the loop variable in for loops. The following example uses implicit typing in a for statement.

  ```csharp
  var phrase = "lalalalalalalalalalalalalalalalalalalalalalalalalalalalalala";
  var manyPhrases = new StringBuilder();
  for (var i = 0; i < 10000; i++)
  {
      manyPhrases.Append(phrase);
  }
  Console.WriteLine("tra" + manyPhrases);
  ```

  

- Do not use implicit typing to determine the type of the loop variable in foreach loops. The following example uses explicit typing in a foreach statement.

  ```csharp
  foreach (char ch in laugh)
  {
    if (ch == 'h')
    {
        Console.Write("H");
    }
    else
    {
        Console.Write(ch);
    }
  }
  Console.WriteLine();
  ```

  

- Avoid the use of var in place of dynamic.