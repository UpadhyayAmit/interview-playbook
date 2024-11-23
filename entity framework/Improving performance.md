---
icon: note
tags: [guide]
---

# Improving performance

Improving performance in Entity Framework (EF) can significantly enhance your application's efficiency. Here are some key strategies:

1. **Use AsNoTracking for Read-Only Queries**: This method disables change tracking, which can reduce overhead for read-only operations[1](https://www.youtube.com/watch?v=jgESld7U5Bw).

2. **Optimize Query Loading**:

   - **Eager Loading**: Use the `Include` method to load related entities in a single query.
   - **Lazy Loading**: Load related entities only when needed, but be cautious as it can lead to multiple database calls.
   - **Explicit Loading**: Manually load related entities when necessary[2](https://learn.microsoft.com/en-us/ef/core/performance/).

3. **Avoid Cartesian Explosion**: When loading related entities, use the `AsSplitQuery` method to avoid large, inefficient joins[1](https://www.youtube.com/watch?v=jgESld7U5Bw).

4. **Efficient Querying**:

   - **Limit Data Retrieval**: Only select the properties you need using the `Select` method.
   - **Pagination**: Implement pagination to handle large datasets efficiently[3](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying).

5. **Use Raw SQL for Complex Queries**: For complex queries, consider using `FromSqlRaw` or `FromSqlInterpolated` to execute raw SQL[1](https://www.youtube.com/watch?v=jgESld7U5Bw).

6. **Database Context Pooling**: Reuse database context objects to reduce the overhead of creating new contexts[1](https://www.youtube.com/watch?v=jgESld7U5Bw).

7. **Indexing**: Ensure your database is properly indexed to speed up query execution[3](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying).

8. **Caching**: Cache frequently accessed data to reduce database load[2](https://learn.microsoft.com/en-us/ef/core/performance/).

By implementing these strategies, you can significantly improve the performance of your Entity Framework applications. If you have specific scenarios or issues, feel free to share, and I can provide more targeted advice!

Caching frequently accessed data can significantly reduce database load and improve application performance. Here’s how you can implement caching in Entity Framework:

1. **In-Memory Caching with MemoryCache**:

   - **Setup**: First, add the `Microsoft.Extensions.Caching.Memory` package to your project.
   - **Configuration**: Configure the `MemoryCache` in your `DbContext`.

   ```csharp
   using Microsoft.Extensions.Caching.Memory;
   using Microsoft.EntityFrameworkCore;

   public class MyDbContext : DbContext
   {
       private readonly IMemoryCache _memoryCache;

       public MyDbContext(DbContextOptions<MyDbContext> options, IMemoryCache memoryCache)
           : base(options)
       {
           _memoryCache = memoryCache;
       }

       public DbSet<User> Users { get; set; }

       protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
       {
           optionsBuilder.UseMemoryCache(_memoryCache);
       }
   }
   ```

2. **Caching Data**:

   - **Retrieve and Cache Data**: Use the `MemoryCache` to store and retrieve frequently accessed data.

   ```csharp
   public class UserService
   {
       private readonly MyDbContext _context;
       private readonly IMemoryCache _memoryCache;

       public UserService(MyDbContext context, IMemoryCache memoryCache)
       {
           _context = context;
           _memoryCache = memoryCache;
       }

       public List<User> GetTopUsers()
       {
           var cacheKey = "TopUsers";
           if (!_memoryCache.TryGetValue(cacheKey, out List<User> topUsers))
           {
               topUsers = _context.Users.OrderBy(u => u.Name).Take(10).ToList();
               var cacheEntryOptions = new MemoryCacheEntryOptions
               {
                   SlidingExpiration = TimeSpan.FromMinutes(10)
               };
               _memoryCache.Set(cacheKey, topUsers, cacheEntryOptions);
           }
           return topUsers;
       }
   }
   ```

3. **Cache Expiration**:

   - **Sliding Expiration**: The cache entry will expire if it hasn't been accessed for a specified period.
   - **Absolute Expiration**: The cache entry will expire after a fixed period, regardless of access.

   ```csharp
   var cacheEntryOptions = new MemoryCacheEntryOptions
   {
       SlidingExpiration = TimeSpan.FromMinutes(10),
       AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)
   };
   ```

By implementing these caching strategies, you can reduce the number of database queries and improve the overall performance of your application[1](https://www.momentslog.com/development/native-application/entity-framework-core-performance-tuning-optimizing-data-access)[2](https://hackernoon.com/essential-entity-framework-core-tips-how-to-optimize-performance-streamline-queries-and-more). If you have any specific scenarios or further questions, feel free to ask!
