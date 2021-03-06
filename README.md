**Contributing**: Either open [an issue on the github page](https://github.com/bdb-opensource/c-sharp-pitfalls/issues), or fork and send a pull request.


# C#/.NET Pitfalls

## C# Language & Compiler


### Pitfalls

1. **Calls to `GetType()` on a `Nullable<T>` instance always return `T`,** although the type is `Nullable<T>`.
2. **Co/contravariance does not work on value types** (i.e. in interface `ITest<out T>`, if it is used with a `T` that's a value type the `out` keyword has no effect).
3. **Type constraints are ignored when resolving method overloads.** Calls to a method that has overrides where only one matches the current type constraints, are still considered ambiguous.
4. **Array types `T[]` implement `ICollection<T>`** and other incompatible interfaces. Arrays of T (`T[]`) "implement" some intefaces such as `ICollection<T>`, but don't *really* implement them. Instead, the throw `NotImplementedException` on methods they can't implement (such as `Add`).
5. **Arrays are covariant** despite being writable. In other words: you can assign `object[] x = new string[1]`. If you then go ahead and do `x[0] = new Something();` you get an exception. Furthermore, as Jon Skeet explains, array covariance is [not just ugly, but slow too](http://msmvps.com/blogs/jon_skeet/archive/2013/06/22/array-covariance-not-just-ugly-but-slow-too.aspx).
6. You **can't combine getter and setter properties via interface inheritance**. The [compiler sees an ambiguity](http://stackoverflow.com/questions/20412783/why-is-a-property-get-considered-ambiguous-when-the-other-interface-is-set-only) when trying to access a property "MyProp" when inheriting a setter from one interface, and the same property's getter from another interface. Properties are **not** like a pair of methods.
		
		interface IGet { int Value { get; } }
		
		interface ISet { int Value { set; } }
		
		interface IBoth : IGet, ISet { }
		
		class Test
		{
		    public void Bla(IBoth a)
		    {
		        var x = a.Value; // Error: Ambiguity between 'IGet.Value' and 'ISet.Value'
		    }
		}

   [According to Eric Lippert](http://stackoverflow.com/a/20413958/562906) the reason is to simplify the compiler's implementation.
7. [**Comparing a class to an interface with `==` always compiles, even if they are unrelated**](http://stackoverflow.com/questions/14697161/whats-the-reasoning-to-fallback-to-objects-operator-when-one-operand-is-an). Consider two unrelated classes `A` and `B` and variables `a` and `b` of corresponding types. Using `==`, you can't compare `a == b` (fails compilation) - this is expected. Now consider an unrelated interface `IC` and variable `c` - you **can** compare `a == c` despite those types not matching. The reason is that there might be a class `D : A, IC` for which the comparison will make sense. This should be at least a warning (apparently it is in VS >= 2012).
8. [**Default parameters are compile-time substitutions**](http://geekswithblogs.net/BlackRabbitCoder/archive/2011/07/28/c.net-little-pitfalls-default-parameters-are-compile-time-substitutions.aspx), so if you change a parameter's  default value without recompiling dependencies, they will still use the old value.

### You may forget this

1. Struct members cannot be protected (because structs cannot be inherited.)

### Bugs

1. When [comparing (using `==`) a nullable to a non-nullable generic struct](http://stackoverflow.com/questions/16797890/why-are-generic-and-non-generic-structs-treated-differently-when-building-expres) in an expression, you get a runtime error. The [Connect issue](https://connect.microsoft.com/VisualStudio/feedback/details/788793/expression-equal-with-one-nullable-and-one-plain-generic-struct-value-causes-invalidoperationexception#) explains that the bug is fixed in Roslyn, but will probably not be fixed in the current (old) compiler.
2. As explained in pitfalls above, co/contravariance does not work for value-type type parameters. However, [the compiler will allow you to constrain a type parameter to be a value (`struct`)](http://stackoverflow.com/questions/9353293/c-sharp-variance-annotation-of-a-type-parameter-constrained-to-be-value-type) resulting in a useless co/contravariance declaration.

## .NET

1. The [`DateTime.Equals` method](http://msdn.microsoft.com/en-us/library/635d5466%28v=vs.110%29.aspx) ignores `.Kind`, which makes the Equals method an unreliable of comparing `DateTime`s. [As discussed in this SO question](http://stackoverflow.com/questions/6930489/safely-comparing-local-and-universal-datetimes), the moral is not to use it at all.

## Working with SQL (including Linq to SQL / Entity Framework)

1. **Bug** ([fixed in EF6](http://data.uservoice.com/forums/72025-ado-net-entity-framework-ef-feature-suggestions/suggestions/1015361-incorrect-handling-of-null-variables-in-where-cl?ref=title%23suggestion-1015361)): A LINQ query such as `where entity.str == myString` when `myString` is null, will always return `false` - even if 'str' is a nullable column. (http://stackoverflow.com/questions/8090894/linq-to-entities-and-null-strings)
	This is because in SQL, `[Column] = null`  always evaluates to false (to check for nulls you need `[Column] is null`).
2. [**DateTime loss of milliseconds resolution**](http://stackoverflow.com/questions/7823966/milliseconds-in-my-datetime-changes-when-stored-in-sql-server): The sql server type `datetime` has less resolution than the .NET `DateTime` type. This means that if you save a DateTime value it won't be the same value when you read it back. EF is aware of this **if you use '2005' for the SqlProvider manifest token** which can be [a bit tricky to do in Code First](http://stackoverflow.com/a/12060518/562906).
	
## WCF

1. If a method parameter is not passed when calling a WCF service method, that parameter is initialized to some default value
    http://thorarin.net/blog/post/2010/08/08/Controlling-WSDL-minOccurs-with-WCF.aspx

2. Linq-to-SQL:	Non-collection references between two entities (e.g. Division -> DivisionDeduction), is not serialized (no DataMember attribute is created on either side)
	- See: http://weblogs.asp.net/zeeshanhirani/archive/2008/07/14/entity-refs-not-getting-serialized-with-wcf-service.aspx#6399262

3. **WCF service hosted in IIS**: When defining custom error pages for some error codes, [IIS will mask WCF exceptions](http://noamlewis.wordpress.com/2012/12/03/custom-error-pages-and-wcf-exceptions-in-iis-and-case-of-bad-ui/) and send the custom error page itself (despite this being a service).
	



