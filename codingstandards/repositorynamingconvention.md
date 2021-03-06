# Repository Naming Convention
All data access should be done through a repository class.  These classes should follow a specific
naming convention to help callers better understand the functionality available.

## Interface Definition
Any repository class should be implementing an interface.  This allows the callers to use dependency
injection with more test-friendly implementations.  The interface should be named with the "I" prefix according
to the coding standard, followed by the primary entity that it manages for its CRUD operations.

## Method Names
Methods for CRUD operations follow very consistent patterns, so it is extremely helpful to the callers of
our repository classes to have consistent names, indicating what the behavior is exactly.

Below is an example to reference as we explain the naming convention.

    // Interface for the persistence layer of orders.
    public interface IOrder
    {
       Order FetchOrder(string number); // Throws exception if number does not exist.
       Order GetLastOrder(string customerNumber); // Safe if no order exists for customerNumber.
       IList<Order> RetrieveOrders(string customerNumber);
       void CreateOrder(Order order); // Inserts only.
       void UpdateOrder(Order order); // Updates only.
       void SaveOrder(Order order); // Inserts or updates appropriately.
       void RemoveOrder(string number);
       IList<Order> SearchOrders(
          DateTime firstOrderDate, DateTime lastOrderDate,
          byte lowestStatus = 20, byte highestStatus = 99,
          string salesRepCode = null)
    }

It is important to note that you will **never have all** of the above methods.  In the case of orders,
for example, we may support `SaveOrder`.  If that's the case, we would not also have `CreateOrder`
and `UpdateOrder` as they are unnecessary and may just cause confusion to the developer.

### Fetch
The name of `Fetch` is to be used when **one and only one** record is expected.  It must always accept
a unique identifier as its parameter(s).  The only time it would not have parameters is if it is
fetching a singleton.  If a record that matches the provided identifier is not found,
a `RecordNotFoundException` is returned.

Order numbers, customer numbers, and product IDs should use a "fetch" because such an identifier is generated by the system and is expected to be there.
In short, `Fetch` is used when you are *expected* to have **one and only one** record.

### Get
The name of `Get` differs from `Fetch` only in that it expects **one or zero** records.
That is, it returns `null` if no record with the provided identifier is found,
whereas `Fetch` throws a `RecordNotFoundException`.

Email addresses or phone numbers may make sense for a "get" because the identifier is natural and outside of our system.  Other examples of a get is if we wanted to apply more logic, such as get last payment for a customer to display on their account management.  It's perfectly normal that they've never made a payment yet.  In short, `Get` is used when it is perfectly normal and expected to have **zero records**.

### Retrieve
`Retrieve` is the name used for getting a list of objects containing **zero or more** members.
It should never return `null`, but rather an empty list.  `Retrieve` is expected to return a list
of manageable size, usually looking up the objects by their relationship to another.  In the above example,
we are returning a list of orders for a given customer.  There is a natural relationship between the order
and customer, so providing a required customerNumber in this case makes sense.

It is possible to have multiple `Retrieve` methods if there are varying relationships, but this is rare.
In most of these cases, you would probably need a `Search`.

### Create
The `Create` method is used only in the case that the caller knows this should be a new object.
If a natural identifier is available, then this method will throw a `DuplicateRecordException`
if the identifier is already defined in the system.

For more information, see [General Data Modification](#general-data-modification).

### Update
The `Update` method is used only in the case that the caller knows this object should
already exist.  Since the caller is explicitly requesting an update, the type must have a natural identifier.
If that identifier is not found, then this method will throw a `RecordNotFoundException`.

For more information, see [General Data Modification](#general-data-modification).

### Save
The `Save` method combines the logic of `Create` and `Update`,
essentially performing the `Create` if it doesn't exist, and `Update` if it does.
As such, `Save` is only appropriate when dealing with objects that have a natural identifier.

For more information, see [General Data Modification](#general-data-modification).

### Remove
`Remove` methods need to accept a unique identifier as the parameter(s).  It should truly remove the object
from the perspective of the caller, and never return it again in any of the reading methods,
such as `Fetch`, `Get`, `Retrieve`, or `Search`.  It should be treated as an idempotent method such as
the `DELETE` method for http.  That is, repeated calls are safe and always return a success, even if it
were previously deleted.

### Search

<span style="color:orange;font-style:italic;font-weight:bold;">NEEDS FURTHER DEFINITION</span>

`Search` methods can be very simple, or very complex, depending on the number of possible records,
the number of optional parameters, and whether pagination is necessary.  Because of this, the standard
on what the signature should look like and proper interaction with it is yet to be defined.

### General Data Modification
There are 3 main methods for data modification: `Create`, `Update`, and `Save`.
If the type modified in these methods contains system-generated values,
such as IDs or time stamps in the database, then this method can return a new copy
of the type with the new values.  Otherwise, it should just be a void return type.
In implementing a returned copy of the type, however, the stored procedure should
just use output parameters for the 1-3 new values rather than returning a result set.
The data delegate can use the input object for the other values.

## Method Names across Layers
Here's a quick comparison for the names from HTTP down to SQL.

<table>
  <tr>
    <th colspan=3>HTTP</th>
    <th colspan=2>Repository Class</th>
    <th>Database</th>
  <tr>
  <tr>
    <th>Method</th>
    <th>Success Code</th>
    <th><em>Not Found</em> Code</th>
    <th>Method Name</th>
    <th><em>Not Found</em> Result</th>
    <th>Stored Procedure Name</th>
  <tr>
  <tr>
    <td>GET</td>
    <td>200</td>
    <td>404</td>
    <td>Fetch</td>
    <td><code>RecordNotFoundException</code></td>
    <td>Fetch</td>
  <tr>
  <tr>
    <td>GET</td>
    <td>200</td>
    <td>204</td>
    <td>Get</td>
    <td><code>null</code></td>
    <td>Fetch with one-row result <br/> or Get with output parameters</td>
  <tr>
  <tr>
    <td>GET</td>
    <td>200 <br/><em>returns list</em></td>
    <td>404</td>
    <td>Retrieve</td>
    <td><em>Empty List</em></td>
    <td>Retrieve</td>
  <tr>
  <tr>
    <td>GET</td>
    <td>200 <br/><em>returns list</em></td>
    <td>204</td>
    <td>Search</td>
    <td><em>Empty List</em></td>
    <td>Search</td>
  <tr>
  <tr>
    <td>POST<br/><em>with unique identifier</em></td>
    <td>201</td>
    <td><em>N/A</em><br/>If already exists, 409?</td>
    <td>Create</td>
    <td><em>N/A</em><br/>If already exists, <code>DuplicateRecordException</code></td>
    <td>Create</td>
  <tr>
  <tr>
    <td>POST<br/><em>with no identifier</em></td>
    <td>201</td>
    <td><em>N/A</em></td>
    <td>Create</td>
    <td><em>N/A</em></td>
    <td>Create</td>
  <tr>
  <tr>
    <td>PUT</td>
    <td>200</td>
    <td>404</td>
    <td>Update</td>
    <td><code>RecordNotFoundException</code></td>
    <td>Update</td>
  <tr>
  <tr>
    <td>PUT</td>
    <td>200 (save), 201 (create)</td>
    <td><em>N/A</em></td>
    <td>Save</td>
    <td><em>N/A</em></td>
    <td>Save</td>
  <tr>
  <tr>
    <td>DELETE</td>
    <td>200</td>
    <td><em>None</em></td>
    <td>Remove</td>
    <td><em>None</em></td>
    <td>Remove</td>
  <tr>
</table>
