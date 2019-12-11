---
title: IEqualityComparer trick
date: 2012-08-29T09:55:00Z
summary: More about Distinct
---

In a previous post I showed that Distinct  uses GetHashCode to spot different values and used GroupBy to workaround.
But this isn’t actually completely true.
The code for Distinct looks like this…
```csharp
public static IEnumerable<TSource> Distinct<TSource>(this IEnumerable<TSource> source, IEqualityComparer<TSource> comparer)
{
  if (source == null)
    throw Error.ArgumentNull("source");
  else
    return Enumerable.DistinctIterator<TSource>(source, comparer);
}

private static IEnumerable<TSource> DistinctIterator<TSource>(IEnumerable<TSource> source, IEqualityComparer<TSource> comparer)
{
  Set<TSource> set = new Set<TSource>(comparer);
  foreach (TSource source1 in source)
  {
    if (set.Add(source1))
      yield return source1;
  }
}
```

So if it can add the item to a Set<T> then item will be returned. So duplicate items are ignored. Quite sneaky!

Notice the Set is created with a comparer. Inside set.Add it looks like this…
```csharp
private bool Find(TElement value, bool add)
{
  int hashCode = this.InternalGetHashCode(value);
  for (int index = this.buckets[hashCode % this.buckets.Length] - 1; index >= 0; index = this.slots[index].next)
  {
    if (this.slots[index].hashCode == hashCode && this.comparer.Equals(this.slots[index].value, value))
      return true;
  }
  // other stuff trimmed
}
```

It’s the if statement that is interesting. What it implies is that if the hash code is the same as another entry then it will separate the two values using comparer.Equals.

What if we supply a comparer that always returns a fixed value for GetHashCode (like 0). Then it will always have to use Equals.
```csharp
public class AlwaysEqualsEqualityComparer<T> : IEqualityComparer<T>
{
    public AlwaysEqualsEqualityComparer(Func<T, T, bool> comparer)
    {
        this.comparer = comparer;
    }

    private readonly Func<T, T, bool> comparer;

    public bool Equals(T x, T y)
    {
        return comparer(x, y);
    }

    public int GetHashCode(T obj)
    {
        return 0;
    }
}
```

So, given I have a collection of these…

```csharp
public class MyThing
{
  public string ID { get; set; }
  public string Name { get; set; }
  public string Type { get; set; }
  public string Colour { get; set; }
}
```
I can use the new comparer like this…

```csharp
var distinctList = thingList.Distinct(new AlwayEqualsEqualityComparer<MyThing>((x,y) => x.Colour == y.Colour);
```
…and get a new collection of Colours.


Here’s the caveat… You should not use this with large collections.

Sets (and other collections) use a system of “buckets” (generally implemented with arrays) to store the items. Which array your item is in is determined with the hash code. You can see some of this in the for statement of the Set.Find method above. So if every item returns the same hash code, every item will go in the same bucket. Scanning a bucket for a duplicate value will get slower the large the bucket.

So only use this trick when you know that the source collection is small (say less than a few thousand).