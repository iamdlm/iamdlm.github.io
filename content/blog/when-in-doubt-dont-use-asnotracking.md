---
title: "When in Doubt, Don't Use AsNoTracking"
date: 2023-10-08T15:38:24+01:00
draft: false
---
In the last couple of months I spent countless hours helping different coworkers debug Entity Framework related errors, and a big chunk of them was because of Entity Framework (EF) change tracking misuse.

## Tracking in Entity Framework
One of the features of EF is change tracking, where EF keeps track of changes made to the entities retrieved from the database. This tracking is essential for EF to perform CRUD (Create, Read, Update, Delete) operations and maintain a connection between the in-memory objects and the database.

When you retrieve an entity from the database using EF, by default, EF tracks changes made to that entity. This means that if you modify the entity and then save it back to the database, EF will generate SQL statements to update only the properties that have changed.

## `AsNoTracking` in Entity Framework
The `AsNoTracking` method in Entity Framework allows you to indicate that you don't want EF to track the changes made to entities. In other words, it tells EF not to include the entities in its change tracking mechanism. This can be useful in various scenarios:

1. **Read-Only Operations**:
If you're performing read-only operations and you have no intention of updating the data, you can use `AsNoTracking` to improve performance and reduce memory usage. This is because EF doesn't need to keep track of changes for entities that won't be modified.

{{< highlight csharp >}}
using (var context = new MyDbContext())
{
    var products = context.Products.AsNoTracking().ToList();
    // products is a list of Product entities without change tracking
}
{{< /highlight >}}

2. **Read-Heavy Scenarios**:
In scenarios where you read a lot of data but update it infrequently, using `AsNoTracking` can help improve performance. This can be beneficial in web applications or APIs where you frequently retrieve data but don't need to track changes.

{{< highlight csharp >}}
using (var context = new MyDbContext())
{
    var orders = context.Orders.AsNoTracking().Where(o => o.CustomerId == customerId).ToList();
    // orders is a list of Order entities without change tracking
}
{{< /highlight >}}

## When `AsNoTracking` Shouldn't Be Used:
1. **Update Operations**: 
If you plan to update entities and save changes back to the database, it's not advisable to use `AsNoTracking` because EF won't be able to track changes.

{{< highlight csharp >}}
using (var context = new MyDbContext())
{
    var product = context.Products.AsNoTracking().FirstOrDefault(p => p.Id == productId);
    product.Name = "Updated Name"; // This change won't be tracked
    context.SaveChanges(); // This won't update the database
}
{{< /highlight >}}

2. **Complex Queries**: 
For complex queries involving multiple related entities that require change tracking, using `AsNoTracking` may lead to issues when updating or saving related data.

3. **Long-Lived Contexts**: 
In scenarios where the DbContext has a long lifespan (e.g., in a Windows service or a long-running application), using `AsNoTracking` excessively can lead to potential memory issues because entities are never released from memory.

## Performance Considerations:
Using `AsNoTracking` can improve performance for read-heavy operations because EF doesn't spend time tracking changes. However, the performance gain can vary depending on the specific use case and the size of the dataset. **It's essential to profile and measure performance improvements** to ensure that using `AsNoTracking` is the right choice for your application.

The source code for this example is available [here](https://github.com/iamdlm/asnotracking-performance), in case you want to play around.