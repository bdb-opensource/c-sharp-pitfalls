# C#/.NET Pitfalls

## C# Compiler


### Pitfalls

1. `GetType()` on a `Nullable<T>` will always return `typeof(T)` (although the type is `Nullable<T>`)
2. Co/contravariance does not work on value types (i.e. in interface `ITest<out T>`, if it is used with a `T` that's a value type the `out` keyword has no effect)
3. Calls to a method that has overrides where only one matches the current type constraints, are still considered ambiguous
4. Arrays of T (`T[]`) "implement" some intefaces such as `ICollection<T>`, but don't *really* implement them. Instead, the throw `NotImplementedException` on methods they can't implement (such as `Add`).
5. The [compiler sees an ambiguity](http://stackoverflow.com/questions/20412783/why-is-a-property-get-considered-ambiguous-when-the-other-interface-is-set-only) when trying to access a property "MyProp" when inheriting a setter from one interface, and the same property's getter from another interface. Properties are **not** like a pair of methods.
		
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

### You may forget this

1. Struct members cannot be protected (because structs cannot be inherited.)

### Bugs

1. When [comparing (using `==`) a nullable to a non-nullable generic struct](http://stackoverflow.com/questions/16797890/why-are-generic-and-non-generic-structs-treated-differently-when-building-expres) in an expression, you get a runtime error. The [Connect issue](https://connect.microsoft.com/VisualStudio/feedback/details/788793/expression-equal-with-one-nullable-and-one-plain-generic-struct-value-causes-invalidoperationexception#) explains that the bug is fixed in Roslyn, but will probably not be fixed in the current (old) compiler.

## .NET

1. The [`DateTime.Equals` method](http://msdn.microsoft.com/en-us/library/635d5466%28v=vs.110%29.aspx) ignores `.Kind`, which makes the Equals method an unreliable of comparing `DateTime`s. [As discussed in this SO question](http://stackoverflow.com/questions/6930489/safely-comparing-local-and-universal-datetimes), the moral is not to use it at all.

## Linq to SQL / Entity Framework

1. A LINQ query such as `where entity.str == myString` when `myString` is null, will always return `false` - even if 'str' is a nullable column. (http://stackoverflow.com/questions/8090894/linq-to-entities-and-null-strings)
	This is because in SQL, `[Column] = null`  always evaluates to false (to check for nulls you need `[Column] is null`).
	
## WCF

1. If a method parameter is not passed when calling a WCF service method, that parameter is initialized to some default value
    http://thorarin.net/blog/post/2010/08/08/Controlling-WSDL-minOccurs-with-WCF.aspx

2. Linq-to-SQL:	Non-collection references between two entities (e.g. Division -> DivisionDeduction), is not serialized (no DataMember attribute is created on either side)
	- See: http://weblogs.asp.net/zeeshanhirani/archive/2008/07/14/entity-refs-not-getting-serialized-with-wcf-service.aspx#6399262

	



