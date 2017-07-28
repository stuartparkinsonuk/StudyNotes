#Azure Mobile Apps

[Develop Cloud Connected Mobile Apps with Xamarin and Microsoft Azure](https://adrianhall.github.io/develop-mobile-apps-with-csharp-and-azure/chapter1/firstapp_pc/)

##Models
The models on the server and the models on the mobile device mirror each other, but are different.

On the server, a model derives from `EntityData` (which derives from `ITableData`) which are defined by the Azure Mobile Apps Server SDK. These provide the background fields needed for mobile sync, namely

``` c#
public string Id { get; set; }
public byte[] Version { get; set; }
public DateTimeOffset? CreatedAt { get; set; }
public DateTimeOffset? UpdatedAt { get; set; }
public bool Deleted { get; set; }
```

These fields have to be included in the mobile client version of the model, and can be included in an abstract base class from which table data models derive.

##Abstractions

On the client we have `TableData` abstract class which provides the necessary background fields necessary to implement cloud sync.

``` c#
public abstract class TableData
{
    public string Id { get; set; }
    public DateTimeOffset? UpdatedAt { get; set; }
    public DateTimeOffset? CreatedAt { get; set; }
    public byte[] Version { get; set; }
}
```

`ICloudTable` provides a CRUD interface into a table.

``` c#
public interface ICloudTable<T> where T : TableData
{
    Task<T> CreateItemAsync(T item);
    Task<T> ReadItemAsync(string id);
    Task<T> UpdateItemAsync(T item);
    Task DeleteItemAsync(T item);

    Task<ICollection<T>> ReadAllItemsAsync();
}
```

`ICloudService` is used for initializing the connection and getting a table definition.

``` c#
public interface ICloudService
{
    ICloudTable<T> GetTable<T>() where T : TableData;
}
```


