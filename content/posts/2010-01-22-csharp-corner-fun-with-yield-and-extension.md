+++
title = "C# Corner - fun with `yield` and extension methods"
date = 2010-01-22T10:11:00-05:00
+++

All .NET developers learn about `IEnumerable` and `IEnumerator `in their first day of development, whether they realize it or not. Every `foreach `statement written utilizes the `.GetEnumerator()` call to iterate over a collection of objects. The "For Each" concept is a really nice piece of syntactic sugar. However, one of its limitations is that it doesn't give you a counter to track which loop iteration you're on. If you want to do that, you have to keep track of it yourself in your own variable. Not a big deal really, just a fact of programming life. This manifests itself in two ways - you can write a `for;;` loop which has the counter built in and then you get the object you want by indexing the collection as shown here:

```csharp
var names = new List<string> { "You", "Me", "Dupree" };

// for;; loop
Console.WriteLine("Method 1:");
for (var i = 0; i < names.Count; i++) {
    var name = names[i];
    Console.WriteLine("{0}-{1}", i, name);
}
```

This method isn't bad, but it has a fatal flaw - it only works on collections that are fully populated. In other words, the collection has to be indexable and cannot be lazy loaded. If you have a collection that dynamically loads or calculates its next value, you have to do something much more kludgy. You must maintain a variable external to your loop that keeps track of your current item index as shown here:

```csharp
var names = GetLazyLoadedNames();

// outside variable
Console.WriteLine("Method 2:");
var j = 0;
foreach (var name in names) {
    Console.WriteLine("{0}-{1}", j, name);
    j++;
}

```

This method works on every `IEnumerable` collection, and is a good general purpose tactic that every .NET dev will need in their toolkit. But, it has the frustrating consequence that it requires a variable outside the scope of the loop, and takes a couple extra lines of code to accomplish. Both of which adversely affect the signal-to-noise ratio of your code.

As a thought experiment, I wondered if I could get the clarity of a simple and tight `foreach` loop while still maintaining the index. It turns out it's really simple (in C# at least) to accomplish with a small immutable helper class and an extension method. Here's the code:

```csharp
public static class IEnumerableExtensions {
    public static IEnumerable<IndexedItem<T>> GetIndexedEnumerator<T>(this IEnumerable<T> me) {
        var index = 0;
        foreach (T item in me) {
            yield return new IndexedItem<T>(item, index);
            index++;
        }
    }
}

public class IndexedItem<T> {
    private T _item;
    private int _index;

    public T Item { get { return _item; } }
    public int Index { get { return _index; } }

    public IndexedItem(T item, int index) {
       _item = item;
       _index = index;
    }
}
```

Now, if you import the namespace containing this extension method, every `IEnumerable` collection you have gets a method called `.GetIndexedEnumerator()` which means that instead of iterating over the objects in the collection, you iterate over a new container object called `IndexedItem` that has the current item in the collection as well as the index of that item. This means that now your `foreach` code can look like this:

```csharp
var names = GetLazyLoadedNames();

// extension method
Console.WriteLine("Method 3:");
foreach (var name in names.GetIndexedEnumerator()) {
    Console.WriteLine("{0}-{1}", name.Index, name.Item);
}
```

This definitely isn't a revolutionary improvement by any means. In fact, in exchange for clarity, you likely will see an ever-so-slightly reduced performance for very large collections since you have the added overhead of the IndexedItem class. Also, unfortunately due to the lack of a 'yield' statement, VB devs (like myself) have a whole lot more work to do to implement this solution. But overall, this is just one more example of why I love .NET. The framework and languages are continually evolving, and while some purists may scoff at the syntactic sugar, I say it's all syntactic sugar over top of zeros and ones, and I have a serious sweet tooth.
