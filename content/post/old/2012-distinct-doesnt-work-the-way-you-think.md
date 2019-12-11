---
title: Distinct - Doesn’t work the way you think it does
date: 2012-08-29T09:55:00Z
summary: IEqualityComparer<T> is a “hash code provider”
---

I recently wanted to retrieve the set of unique values out of a list of objects. Say the object looked like this…

```csharp
public class MyThing
{
  public string ID { get; set; }
  public string Name { get; set; }
  public string Type { get; set; }
  public string Colour { get; set; }
}
```

I have 100 of these in a List<MyThing>. How do I get a list of unique Type/Colour combinations. You might think that the obvious way would be to use the Distinct linq operator. It gets a little complicated because you need to define an IEqualityComparer<MyThing> to define which objects are equal (ie where Type and Colour are the same). So you need one of these…

```csharp
public class MyThingComparer : IEqualityComparer<MyThing>
{
  public bool Equals(MyThing x, MyThing y)
  {
    // Object.ReferenceEquals and null check left out for clarity

    return x.Type == y.Type && x.Colour == y.Colour;
  }

  public int GetHashCode(MyThing obj)
  {
    return obj.GetHashCode();
  }
}
```

Seems straight forward enough. We’re checking that the Type and Colour properties are the same in two objects. So we try…
```csharp
var distinctList = thingList.Distinct(new MyThingComparer());
```
…expecting a nice short list of unique Type/Colour combinations. What we actually get is the whole list. Much head scratching and ranting ensues.

Then you run into a post from 2008 http://blog.jordanterrell.com/post/LINQ-Distinct()-does-not-work-as-expected.aspx and you learn that it isn’t using you’re Equals function. It’s using GetHashCode to determine the objects with the same hash code are equal. Once you calm down and think about it, it kind of makes sense. But is a little counter intuitive. To quote from the MSDN page for [Object.GetHashCode](https://docs.microsoft.com/en-us/dotnet/api/system.object.gethashcode) …

  >A hash code is a numeric value that is used to identify an object during equality testing.

It goes on to describe IEqualityComparer<T> as a “hash code provider”.

So we could try to figure out some algorithm to calculate a hash code that is equal and unique for our combination of property values. But this is incredibly difficult to get provably correct.

There is another way…
```csharp
var distinctList = thingList.GroupBy(x => new { x.Type, x.Colour }).First();
```

We’re using an anonymous object to group by the required properties then taking the first out of the group.
Yay, it works! But let’s dig a little.

[Enumerable.GroupBy](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupby) says…
> returns a collection of [IGrouping<TKey, TElement>](https://docs.microsoft.com/en-us/dotnet/api/system.linq.igrouping-2) objects, one for each distinct key that was encountered.

> The default equality comparer Default is used to compare keys.

Oops, there’s that “distinct” word again. Read a little further and we discover that this “Default” thing is a IEqualityComparer that uses Object.GetHashCode. Aren’t we back where we started? Why does the GroupBy seem to work as expected? A little further research leads to http://odetocode.com/blogs/scott/archive/2008/03/25/and-equality-for-all-anonymous-types.aspx Where the summary states…

> Turns out the C# compiler overrides Equals and GetHashCode for anonymous types. The implementation of the two overridden methods uses all the public properties on the type to compute an object's hash code and test for equality. If two objects of the same anonymous type have all the same values for their properties – the objects are equal. This is a safe strategy since anonymously typed objects are essentially immutable (all the properties are read-only). Fiddling with the hash code of a mutable type gets a bit dicey.

So an anonymous type implements GetHashCode as a combination of each of the properties of the type. So we can rely on a bunch of clever engineers to figure out how to figure out the algorithm.

Which leads us back to the original need

```csharp
var distinctList = thingList.Select(x => new { x.Type, x.Colour }).Distinct();
```