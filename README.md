# C#/.NET Pitfalls

## C# Compiler

1. GetType() on Nullable<T> will always return typeof(T)
2. co/contra variance does not work on value types (i.e. in interface ITest<out T>, if it is used with a T that's a value type the 'out' keyword has no effect)
3. calls to a method that has overrides where only one matches the current type constraints, are still considered ambiguous
4. Arrays of T (`T[]`) "implement" some intefaces such as `ICollection<T>`, but don't *really* implement them. Instead, the throw `NotImplementedException` on methods they can't implement (such as `Add`).

### You may forget this

1. Struct members cannot be protected (because structs cannot be inherited.)

## Linq to SQL / Entity Framework

1. A LINQ query such as `where entity.str == myString` when `myString` is null, will always return `false` - even if 'str' is a nullable column. (http://stackoverflow.com/questions/8090894/linq-to-entities-and-null-strings)
	This is because in SQL, `[Column] = null`  always evaluates to false (to check for nulls you need `[Column] is null`).

## .NET

1. The [`DateTime.Equals` method](http://msdn.microsoft.com/en-us/library/635d5466%28v=vs.110%29.aspx) ignores `.Kind`, which makes the Equals method an unreliable of comparing `DateTime`s. [As discussed in this SO question](http://stackoverflow.com/questions/6930489/safely-comparing-local-and-universal-datetimes), the moral is not to use it at all.
	
## WCF

1. If a method parameter is not passed when calling a WCF service method, that parameter is initialized to some default value
    http://thorarin.net/blog/post/2010/08/08/Controlling-WSDL-minOccurs-with-WCF.aspx

2. Linq-to-SQL:	Non-collection references between two entities (e.g. Division -> DivisionDeduction), is not serialized (no DataMember attribute is created on either side)
	- See: http://weblogs.asp.net/zeeshanhirani/archive/2008/07/14/entity-refs-not-getting-serialized-with-wcf-service.aspx#6399262

	



