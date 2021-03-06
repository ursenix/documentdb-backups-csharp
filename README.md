# Azure DocumentDB - C# (6.0) library

Azure DocumentDB, or shortly DocumentDB, is a NoSQL document database service offered by Microsoft. 

This library offers a set of classes to work with DocumentDB using .NET C# 6.0. 

## Backups

The framework also has a DocumentDB history backup solution. The backups will be created in a different collection (in the same database or in a different one). 

The implementation is very simple and will only backup the documents. A complete Microsoft backup solution is already planned.
- http://feedback.azure.com/forums/263030-documentdb

####Service definition:
```csharp
    public class FoodService : DocumentDBService
    {
        public FoodService()
            : base(endPointUrl, authorizationKey, database)
        {
            // Create collection service
            ShoppingList = CreateCollectionService<Item>("ShoppingListCollection").Result;

            // Create a backup service for the shopping list collection
            BackupService = CreateCollectionBackupService(ShoppingList);
        }

        public ICollectionService<Item> ShoppingList { get; }
        public IBackupService BackupService { get; }
    }
```

####Create and restore backups:
```csharp
using (var foodService = new FoodService())
{
    Console.WriteLine("\n> Create some documents:");
    foodService.ShoppingList.CreateDocument(new Item { Name = "Milk", Description = "Skimmed milk" }).Wait();
    foodService.ShoppingList.CreateDocument(new Item { Name = "Milk", Description = "Whole milk" }).Wait();
    foodService.ShoppingList.CreateDocument(new Item { Name = "Water", Description = "Mineral" }).Wait();
    
    foreach (var item in foodService.ShoppingList.AllDocuments)
    {
        Console.WriteLine($"- {item.Name}- {item.Description}- {item.Id}");
    }

    Console.WriteLine("\n> Find all items with name = Milk:");
    foreach (var item in foodService.ShoppingList.Where(d => d.Name.Equals("Milk")))
    {
        Console.WriteLine($"- {item.Name}- {item.Description}- {item.Id}");
    }

    Console.WriteLine("\n> Create a backup:");
    var backup = foodService.BackupService.CreateBackup("my backup").Result;
    Console.WriteLine("Backup created at" + backup.Timestamp);

    Console.WriteLine("\n> Delete water:");
    foreach (var item in foodService.ShoppingList.Where(d => d.Name.Equals("Water")))
    {
        Console.WriteLine($"Delete: {item.Name} - {item.Id}");
        foodService.ShoppingList.DeleteDocument(item.SelfLink).Wait();
    }

    Console.WriteLine("\n> Restore backup:");
    foodService.BackupService.RestoreBackup(backup);
    Console.WriteLine("Backup restored!");

    Console.WriteLine("\n> List all the documents:");
    foreach (var item in foodService.ShoppingList.AllDocuments)
    {
        Console.WriteLine($"- {item.Name}- {item.Description}- {item.Id}");
    }

    Console.WriteLine("\n> Delete database:");
    foodService.ShoppingList.DatabaseService.DeleteDatabase().Wait();
}
 ```

#### Class diagram
![alt tag](https://cloud.githubusercontent.com/assets/9339027/9705124/71f3036a-54ba-11e5-959a-4683916ccefe.png)


Read more on my blog: [http://softwarejuancarlos.com](http://softwarejuancarlos.com)  
