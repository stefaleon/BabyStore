# BabyStore

## 1

### Using the Optional URL ID Parameter

To complete this very simple overview of the default routing setup, we’ll alter the site to use the optional ID
parameter.

Alter the About method of the Controllers\HomeController.cs file so that it reads as follows:
```
public ActionResult About( string id )
{
ViewBag.Message = "Your application description page. You entered the ID " + id;
return View();
}
```
Here we are telling the method to process a parameter passed into it called id . This corresponds with
the name of the optional third parameter in the default route. The ASP.NET MVC model binding system is
used to automatically map the ID parameter from the URL into the id parameter in the About method. We
will cover model binding in more detail later.

Now start the web site without debugging and navigate to the About page. Now add an ID onto the end
of the URL such as http://localhost:58735/Home/About/7 .

This will then process 7 as the value of the ID parameter and pass this to the About method, which then
adds this to the ViewBag for display in the view.


### The Purpose of the Layout Page

#### Tip

If you want a view not to use the layout page specified in the _ ViewStart.cshtml file, add the entry
```
  @{ Layout = null; }
```
to the top of the file.




## 2

### Adding the Model Classes

```
public class Product
{
  public int ID { get; set; }
  public string Name { get; set; }
  public string Description { get; set; }
  public decimal Price { get; set; }
  public int? CategoryID { get; set; }
  public virtual Category Category { get; set; }
}
```
* CategoryID —Represents the ID of the category that the product is assigned to. It
will be set up as a foreign key in the database. We have allowed this to be empty by
setting the type to int? to model the fact that a product does not need to belong to
a category. This is crucial to avoid the scenario where, upon deletion of a category,
all products in that category are also deleted. By default, Entity Framework enables
cascade deletes for on-nullable foreign keys, meaning that if the CategoryID was not
nullable, then all the products associated with a category would be deleted when the
category was deleted.

* Category —A navigation property . Navigation properties contain other entities that
relate to this entity so in this case this property will contain the category entity that
the product belongs to. If a navigation property can hold multiple entities, it must be
defined as a list type. Typically the type used is ICollection . Navigation properties
are normally defined as virtual so that they can be used in certain functionality such
as with Lazy Loading.

```
public class Category
{
  public int ID { get; set; }
  public string Name { get; set; }
  public virtual ICollection<Product> Products { get; set; }
}
```

* Products —A navigational property that will contain all the product entities belonging to a category


### Adding a Database Context

The database context is the main class that coordinates Entity Framework functionality for a data model.
Create a new folder in the project named DAL by right-clicking the project ( BabyStore ) in Solution
Explorer. Click Add then New Folder. Now add a new class named StoreContext.cs to the new DAL folder.
Update the code in the StoreContext class as follows:

```
using BabyStore.Models;
using System.Data.Entity;

namespace BabyStore.DAL
{
  public class StoreContext:DbContext
    {
      public DbSet<Product> Products { get; set; }
      public DbSet<Category> Categories { get; set; }
    }
}
```

The context class derives from the class System.Data.Entity.DbContext and there is typically one
database context class per database, although in more complex projects it is possible to have more. Each
DbSet property in the class is known as an Entity Set and each typically corresponds to a table in the
database; for example the Products property corresponds to the Products table in our database. The code
DbSet<Product> tells Entity Framework to use the Product class to represent a row in the Products table.


### Specifying a Connection String

Now we have a database context and some model classes, so we need to tell Entity Framework how to connect
to the database. Add a new entry to the connectionStrings section of the main Web.Config file as follows:

```
<connectionStrings>
  <add name="DefaultConnection" connectionString="Data Source=(LocalDb)\MSSQLLocalDB;Att
  achDbFilename=|DataDirectory|\aspnet-BabyStore-20160203031215.mdf;Initial Catalog=aspnet-
  BabyStore-20160203031215;Integrated Security=True"
  providerName="System.Data.SqlClient" />
  <add name="StoreContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDb
  Filename=|DataDirectory|\BabyStore.mdf;Initial Catalog=BabyStore;Integrated Security=True"
  providerName="System.Data.SqlClient" />
</connectionStrings>
```

This new entry tells Entity Framework to connect to a database called BabyStore.mdf in the App_Data
folder of our project. I’ve chosen to store the database here so that it can be copied with the project. AttachDb
Filename=|DataDirectory|\BabyStore.mdf; specifies to create the database in the App_Data folder.

An alternative would be to specify the connectionString entry as Data Source=(LocalDB)\
MSSQLLocalDB;Initial Catalog=BabyStore.mdf;Integrated Security=True , which would then create the
database under the user’s folder (normally C:\Users\User on Windows ).

The other existing connectionString entry is used for the database that was created automatically at
the beginning of the project when we chose the authentication option Individual User Accounts. We will
cover the use of this later in the book.

It’s also worth noting that you don’t need to define a connection string in web.config . If you don’t do
so, then Entity Framework will use a default one based on the context class.

■ Note Make sure you update the Web.config file in the root of the project and not the one in the Views folder.
