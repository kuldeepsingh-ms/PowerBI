# Calculate State Time

This article provides a series of recipes using DAX calculations to evaluate time spent by work items in any combination of states. Specifically, you'll learn how to add the following calculated columns and one measure and use them to generate various trend charts.

## Add State Sort Order

By default, Power BI will show states sorted alphabetically in a visualization. This can be misleading when you want to visualize time in state and `New` state shows up after `Active` state.

To resolve this issue, we will create a new calculated column for states and assign the order as per our requirement. Follow below steps to create a new column for states.

1. To add the `State Sort Order` calculated column, from the `Modeling` tab, choose `New Column`.
2. Replace the default text with the following code.
```
  State Sort Order = 
  SWITCH ( 
        '<ViewName>'[State], 
          "New", 1, 
          "Active", 2, 
          "Resolved", 3, 
          "Closed", 4, 
          "Removed",5, 
          6
      )
```
3. Update your desired states in above peice of code and click the checkmark.
4. From the `Modeling` tab, choose `Sort by Column` and then select the `State Sort Order field`.

## Add Date Previous
The next step for calculating time-in-state requires mapping the previous interval (day, week, month) for each row of data in the dataset.

To add the `Date Previous` calculated column, from the `Modeling` tab, choose `New Column` and then replace the default text with the following code and click the checkmark.
```
  Date Previous = 
  CALCULATE (
      MAX ( '<ViewName>'[Date] ),
      ALLEXCEPT ( '<ViewName>', '<ViewName>'[Work Item Id] ),
      '<ViewName>'[Date] < EARLIER ( '<ViewName>'[Date] )
    )
```

## Add Date Diff in Days
Date Previous calculates the difference between the previous and current date for each row. With Date Diff in Days, we'll calculate a count of days between each of those periods. For most rows in a daily snapshot, the value will equal 1. However, for many work items which have gaps in the dataset, the value will be larger than 1.

It is important to consider the first day of the dataset where Date Previous is blank. In this example we give that row a standard value of 1 to keep the calculation consistent.

From the `Modeling` tab, choose `New Column` and then replace the default text with the following code and click the checkmark.
```
  Date Diff in Days = 
  IF (
      ISBLANK ( '<ViewName>'[Date Previous] ),
      1,
      DATEDIFF (
            '<ViewName>'[Date Previous],
            '<ViewName>'[Date],
              DAY
            )
     )
```



## Exercise 2 - Create a blank ASP.NET Core MVC application
In this exercise we will create a ASP.NET Core MVC application with controllers and views.

You can refer ["Microsoft Docs - Get started with ASP.NET Core MVC"](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/start-mvc) for this excercise.

## Exercise 3 - Set up the ASP.NET Core MVC application

### Steps
Now that we have most of the ASP.NET Core MVC code that we need for this solution, let's add the NuGet packages required to connect to Azure Storage Account.

1. Add Azure Storage Account .NET NuGet package to your project
    * Right-click your MVC project, **Manage NuGetPackages...**
    * In the NuGet window search for `WindowsAzure.Storage` and install
2. Under Models folder add a new class `ToDoItem.cs`
    ```cs
    using Microsoft.WindowsAzure.Storage.Table;
    using Newtonsoft.Json;

    public class ToDoItem : TableEntity
    {
        [JsonProperty(PropertyName = "name")]
        public string Name { get; set; }

        [JsonProperty(PropertyName = "category")]
        public string Category { get; set; }

        [JsonProperty(PropertyName = "description")]
        public string Description { get; set; }

        [JsonProperty(PropertyName = "isComplete")]
        public bool Completed { get; set; }
    }
    ```

4. Add Views
   * Create a new `Item` folder under `Views` folder.
   * Add below views under `Item` folder.
       * [Index](./Views/Index.cshtml)
       * [Create](./Views/Create.cshtml)
       * [Edit](./Views/Edit.cshtml)
       * [Details](./Views/Details.cshtml)
       * [Delete](./Views/Delete.cshtml)
    
5. Add a new Controller under Controllers folder
    * Name : ItemController
    * MVC Controller - Empty
    

5. Add a new class `IToDoItemService.cs` under Services Folder and replace it's contents with the following code:
    ```cs
    using Models;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    public interface IToDoItemService
    {
        Task AddToDoItem(ToDoItem item);
        Task DeleteToDoItem(string releaseYear, string title);
        Task<ToDoItem> GetDoToItem(string category, string rowKey);
        Task<List<ToDoItem>> GetToDoList();
        Task UpdateToDoItem(ToDoItem item);
    }
    ```
    
6. Add a new class `ToDoItemService.cs` under Services Folder and replace it's contents with the following code:
    ```cs
    using Models;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    public class ToDoItemService : IToDoItemService
    {
        private readonly IAzureTableStorage<ToDoItem> repository;

        public ToDoItemService(IAzureTableStorage<ToDoItem> repository)
        {
            this.repository = repository;
        }

        public async Task<List<ToDoItem>> GetToDoList()
        {
            return await this.repository.GetList();
        }

        public async Task<ToDoItem> GetDoToItem(string category, string rowKey)
        {
            return await this.repository.GetItem(category, rowKey);
        }

        public async Task AddToDoItem(ToDoItem item)
        {
            await this.repository.Insert(item);
        }

        public async Task UpdateToDoItem(ToDoItem item)
        {
            await this.repository.Update(item);
        }

        public async Task DeleteToDoItem(string releaseYear, string title)
        {
            await this.repository.Delete(releaseYear, title);
        }
    }
    ```
    
7. Within the **Startup.cs** file, add the following line to the `ConfigureServices` method:
   ```cs
   services.AddScoped<IAzureTableStorage<ToDoItem>>(factory =>
   {
       return new AzureTableStorage<ToDoItem>(
           new AzureTableSettings(
               storageAccount: Configuration["Table_StorageAccount"],
               storageKey: Configuration["Table_StorageKey"],
               tableName: Configuration["Table_TableName"]));
   });
   services.AddScoped<IToDoItemService, ToDoItemService>();
   ```

8. Define the configuration in Secret file.
    * Right click on project and click on **Manage User Secrets** and add below settings:
    ```json
      {
        "Table_StorageAccount": "<storage account name>",
        "Table_StorageKey": "UXkeWA0h5afXSpyFvud2YdJ9EPR1NCFHFNu7Pcuq1SjHzdf3H7B4D/Tm+2hgpJeFTGnIGuK0GCpGMid8j1TpIQ==",
        "Table_TableName": "storagedemotable"
      }
    ```
    * Replace the account URI and key from the values in the portal. In the Azure portal, in your Azure storage accounts's blade:
      * Navigate to **Access keys**
      * Copy the storage account name replace the value in 'secret.json' for **Table_StorageAccount**
      * Use value of Key from Key1 and replace the value in 'secret.json' for **Table_StorageKey** 

## Exercise 5 - Run and Test your application

### Steps
1. Select F5 in Visual Studio to build the application in debug mode. It should build the application and launch a browser.
2. Click on Items menu tab, it will open a page with empty grid.
3. Select the Create New link and add values to the Name and Description fields. Leave the Completed check box unselected.
4. Select Edit next to an Item on the list. The app opens the Edit view where you can update any property of your object, including the Completed flag.
5. Verify the data in the Azure storage portal using Data Explorer.

## Exercise 6 - Extend your application (Optional)

### Steps
1. Hide the completed items from the grid.
2. Enable your application to accept the Quantity of items.
