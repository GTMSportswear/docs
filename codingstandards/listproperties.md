# List Properties

Many types, especially data structures, will contain properties of a list type,
such as `IList<T>` or `IReadOnlyList<T>`.  List properties should be implemented
under the following guidelines, especially `public` properties.

- The list property itself should be read-only.
- The list property should be initialized to a new instance when the containing data structure is constructed.
- The list property should be of an interface type when possible.  That is, use `IList<T>` rather than `List<T>`.
- Use the read-only collections when appropriate, such as `IReadOnlyCollection<T>`, `IReadOnlyList<T>`, and `IReadOnlyDictionary<TKey,TValue>`.

## Validation

As with any type, validation of the input should be performed as early as possible.
In most cases, this can be accomplished in the constructor of the class.
In the cases of lists, however, when a list is expected to only contain certain values,
validation needs to occur as values are added, inserted, or set using an indexer.

Here is an example of how to **not** validate the entries of a list.
Note the list property, `Attributes`, correctly defined as a read-only property as `IList<string>`.

```csharp
public class Example
{
   public string Property { get; }

   public IList<string> Attributes { get; } = new List<string>();

   public Example(string property)
   {
      Property = property;
   }
}
```

Lets say the `Attributes` list should not contain any null or empty values.
Before performing an operation, we may want to validate the `Attributes` property
so we can throw an appropriate `ArgumentException`.

**Incorrect Approach**

```csharp
public void SaveExample(Example example)
{
   ParameterVerifier.VerifyIsNotNull(example, nameof(example));

   var index = 0;

   // Messy code to give caller meaningful information.
   foreach (var a in example.Attributes)
   {
      if (string.IsNullOrEmpty(a))
         throw new ArgumentException($"Attribute at position {index} is null or empty.");

      index++;
   }

   // Now save Example.
}
```

Again, the desired way to handle this is to throw the exception as early as possible.
The way to accomplish this is to throw an exception as values are added to the `Attributes` list.
The above method, then, can assume all values in `Attributes` have been verified rather than performing its own validation.

One way to accomplish this is to create a custom list for every data structure such as an AttributeList, OrderLineList, and AddressList.
Of course, this can get fairly tedious, and we want that to be the exception rather than the rule.

## The Solution

To solve this problem we are introducing a generic `VerifiedList` that simply accepts a verification delegate, as well as more specific list types of `NonNullList` and `NonEmptyStringList` to handle common scenarios.  Furthermore, there is also an abstract class, `ConstrainedList` from which all the aforementioned types derive and can be used for when some more custom code is needed.

In our above example, we just wanted to ensure `Attributes` did not have any null or empty values.  We simply change the type of the property.

```csharp
public class Example
{
   public string Property { get; }

   public IList<string> Attributes { get; } = new NonEmptyStringList();

   public Example(string property)
   {
      Property = property;
   }
}
```

## Avoid Misuse

What the above solution is **not** is a way to verify the objects themselves within a list.  For example, if my `Order` class will have a list property named `Lines` containing a list of `OrderLine` objects, the contents of `OrderLine` is not verified as each is added to the list, but rather as the `OrderLine` is created, just like any other type.

```csharp
public class OrderLine
{
   public string ProductId { get; }

   public int Quantity { get; }

   public OrderLine(string productId, int quantity)
   {
      ParameterVerifier.VerifyIsNotNullOrEmpty(productId, nameof(productId));
      ParameterVerifier.VerifyIsAtLeast(quantity, 1, nameof(quantity));

      ProductId = productId;
      Quantity = quantity;
   }
}
```

The above `OrderLine` type can then be placed into a list property, but the list does not necessarily need to be constrained.  However, it *may* be appropriate to use the `NonNullList` for order lines, but perform the validation of the order line itself in the constructor of the `OrderLine` type like in the above code block.

```csharp
public class Order
{
   public string CustomerId { get; }

   // Note the use of NonNullList here.
   public IList<OrderLine> Lines { get; } = new NonNullList<OrderLine>();

   public Order(string customerId)
   {
      ParameterVerifier.VerifyIsNotNullOrEmpty(customerId, nameof(customerId));

      CustomerId= customerId;
   }
}
```
