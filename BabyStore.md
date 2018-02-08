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



### Adding a Category Controller and Views

The code

```
private StoreContext db = new StoreContext();
```
instantiates a new context object for use
by the controller. This is then used throughout the lifetime of the controller and disposed of by the Dispose
method at the end of the controller code.



The Index method is used to return a list of all the categories to the Views\Categories\Index.cshtml view:

```
// GET: Categories
public ActionResult Index()
{
  return View(db.Categories.ToList());
}
```

The Details method finds a single category from the database based on the id parameter. As you saw in
Chapter 1 , the id parameter is passed in via the URL using the routing system.

```
// GET: Categories/Details/5
public ActionResult Details(int? id)
{
  if (id == null)
  {
    return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
  }
  Category category = db.Categories.Find(id);
  if (category == null)
  {
    return HttpNotFound();
  }
  return View(category);
}
```

The GET version of the Create method simply returns the Create view. This may seem a little strange at
first but what it means is that this returns a view showing a blank HTML form for creating a new category.

```
// GET: Categories/Create
public ActionResult Create()
{
  return View();
}
```

There is another version of the Create method that’s used for HTTP POST requests. This method is
called when the user submits the form rendered by the Create view. It takes a category as a parameter and
adds it to the database. If it’s successful it returns the web site to the Index view; otherwise, it reloads the
Create view.

```
// POST: Categories/Create
// To protect from overposting attacks, please enable the specific properties you want to bind to, for
// more details see http://go.microsoft.com/fwlink/?LinkId=317598.
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Create([Bind(Include = "ID,Name")] Category category)
{
  if (ModelState.IsValid)
  {
    db.Categories.Add(category);
    db.SaveChanges();
    return RedirectToAction("Index");
  }
  return View(category);
}
```
Because this method is an HTTP POST , it contains some extra code:

*  The [HttpPost] attribute tells the controller that when it receives a POST request for
the Create action, it should use this method rather than the other create method.
* [ValidateAntiForgeryToken] ensures that the token passed by the HTML form,
thus validating the request. The purpose of this is to ensure that the request actually
came from the form it is expected to come from in order to prevent cross-site request
forgeries. In simple terms, a cross-site request forgery is a request from a form on
another web site to your web site with malicious intentions.
* The parameters ([Bind(Include = "ID,Name")] Category category) tell the
method to include only the ID and the Name properties when adding a new category.
The Bind attribute is used to protect against overposting attacks by creating a list
of safe properties to update; however, as we will discuss later, it does not work as
expected and so it is safer to use a different method for editing or creating where
some values may be blank. As an example of overposting, consider a scenario where
the price is submitted as part of the request when a user submits an order for a
product. An overposting attack would attempt to alter this price data by changing the
submitted request data in an attempt to buy the product cheaper.



The GET version of the Edit method contains code identical to the Details method. The method finds a
category by ID and then returns this to the view. The view is then responsible for displaying the category in a
format that allows it to be edited.

```
// GET: Categories/Edit/5
public ActionResult Edit(int? id)
{
  if (id == null)
  {
    return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
  }
  Category category = db.Categories.Find(id);
  if (category == null)
  {
    return HttpNotFound();
  }
  return View(category);
}
```
The POST version of the Edit method is very similar to the POST version of the Create method. It
contains an extra line of code to check that the entity has been modified before attempting to save it to the
database and, if it’s successful, the Index view is returned or else the Edit view is redisplayed:

```
// POST: Categories/Edit/5
// To protect from overposting attacks, please enable the specific properties you want to bind to, for
// more details see http://go.microsoft.com/fwlink/?LinkId=317598.
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Edit([Bind(Include = "ID,Name")] Category category)
{
  if (ModelState.IsValid)
  {
    db.Entry(category).State = EntityState.Modified;
    db.SaveChanges();
    return RedirectToAction("Index");
  }
  return View(category);
}
```

There are also two versions of the Delete method. ASP.NET MVC scaffolding takes the approach of
displaying the entity details to the users and asking them to confirm the deletion before using a form to
actually submit the deletion request. The GET version of the Delete method is shown here. You will notice
this is very similar to the Details method in that it finds a category by ID and returns it to the view:

```
// GET: Categories/Delete/5
public ActionResult Delete(int? id)
{
  if (id == null)
  {
    return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
  }
  Category category = db.Categories.Find(id);
  if (category == null)
  {
    return HttpNotFound();
  }
  return View(category);
}
```

The POST version of the Delete method performs an anti-forgery check. It then finds the category by ID,
removes it, and saves the database changes. See Chapter 4 for how to correct this issue.

```
// POST: Categories/Delete/5
[HttpPost, ActionName("Delete")]
[ValidateAntiForgeryToken]
public ActionResult DeleteConfirmed(int id)
{
  Category category = db.Categories.Find(id);
  db.Categories.Remove(category);
  db.SaveChanges();
  return RedirectToAction("Index");
}
```

This auto-generated Delete method does not work correctly due to the fact that the product entity
contains a foreign key referencing the category entity.

■ Note There are several reasons why ASP.NET takes this approach to disallow a GET request to update
the database and several comments and debates about the different reasons about the security of doing so;
however, one of the key reasons for not doing it is that a search engine spider will crawl public hyperlinks
in your web site and potentially be able to delete all the records if there is an unauthenticated link to delete
records. Later we will add security to editing categories so that this becomes a moot point.



### Adding a Product Controller and Views

The application needs to provide a way to associate a product with a category and it does this by rendering an
HTML select element so that the users can pick a category from a list when creating or editing a product.

In the controller, the new code responsible for this is found in both versions of the Edit methods, as
well as in the POST version of the Create method:

```
ViewBag.CategoryID = new SelectList(db.Categories, "ID", "Name", product.CategoryID);
```

In the GET version of the Create method, the following similar code is used:

```
ViewBag.CategoryID = new SelectList(db.Categories, "ID", "Name");
```

This code assigns an item to a ViewBag property named CategoryID . The item is a SelectList object
consisting of all the categories in the database, with each entry in the list using the Name property as the text
and the ID field as the value. The optional fourth parameter determines the preselected item in the select
list. As an example, if the fourth argument product.CategoryID is set to 2 , then the Toys category will be
preselected in the drop-down list when it appears in the view. Figure 2-9 shows how this appears in a view.

The views display an HTML select element by using the following HTML Helper:

```
@Html.DropDownList("CategoryID", null, htmlAttributes: new { @class = "form-control" })
```

This code generates an HTML element based on the ViewBag.CategoryID property and assigns the CSS
class form control to it. If the string specified in the first argument matches a ViewBag property name, it is
automatically used rather than having to specify a reference to the ViewBag in the DropDownList helper method.





### Splitting DataAnnotations into Another File Using MetaDataType

Some developers prefer the model classes as clean as possible, therefore preferring not to add
DataAnnotations to them. This is achieved by using a MetaDataType class as follows.

Add a new class to the Models folder called ProductMetaData.cs and update the contents of the file to
the following code:

```
using System.ComponentModel.DataAnnotations;
namespace BabyStore.Models
{
  [MetadataType(typeof(ProductMetaData))]
  public partial class Product
  {
  }

  public class ProductMetaData
  {
    [Display(Name = "Product Name")]
    public string Name;
  }
}
```

This declares the Product class as now being a partial class, meaning it is split across multiple files. The
DataAnnotation [MetadataType(typeof(ProductMetaData))] is used to tell .NET to apply metadata to the
Product class from the ProductMetaData class.

Modify the Product class back to its original state but declare it as a partial class so that it can be used in
conjunction with the other class declaration in the ProductMetaData.cs file.

```
namespace BabyStore.Models
{
  public partial class Product
  {
    public int ID { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int? CategoryID { get; set; }
    public virtual Category Category { get; set; }
  }
}
```

The results of this code are exactly the same as in Figure 2-17 ; however, using this code, the Product
class was not altered except to declare it as partial. This can be a useful strategy when working with
classes that have been automatically created that you do not want to alter, for example, when using Entity
Framework Database First.



### A Simple Query: Sorting Categories Alphabetically

Categories in the Categories Index page are currently sorted by the ID property so we’ll change this to sort
them alphabetically by the Name property.
This is a simple change to make. Open the Controllers\CategoriesController.cs file and update the
Index method as follows:

```
// GET: Categories public ActionResult Index()
{
  return View(db.Categories .OrderBy(c => c.Name) .ToList());
}
```



### Filtering Products by Category: Searching Related Entities Using Navigational Properties and Include

You’ve seen how to create a very basic web site showing two lists of different entities. Now we are going to
add some useful functionality and get these lists to interact. We’ll do this using a chosen value from the list of
categories to filter the list of products. To do this, we’ll need to make the following changes to the code:

* Update the Index method in the ProductsController so that it receives a parameter
representing a chosen category and returns a list of products that belong to that
category.
* Transform the list shown in the Category Index Page to a list of hyperlinks that target
the ProductsController Index method rather than a list of text items.

First change the ProductsController Index method as follows:

```
public ActionResult Index( string category )
{
  var products = db.Products.Include(p => p.Category);
  if (!String.IsNullOrEmpty(category))
  {
    products = products.Where(p => p.Category.Name == category);
  }
  return View(products.ToList());
}
```

This code adds a new string parameter named category to the method. An if statement has been
added to check if the new category string is empty. If the category string is not empty, the products are
filtered by using the navigational property Category in the Product class using this code:

```
products = products.Where(p => p.Category.Name == category);
```

The use of the Include method in the following code line is an example what is known as **eager loading**:

```
 var products = db.Products.Include(p => p.Category);
```

It tells Entity Framework to perform a single query and retrieve all the products and also all the related
categories. Eager loading typically results in an SQL join query that retrieves all the required data at once.

You could omit the Include method and Entity Framework would use lazy loading, which would involve
multiple queries rather than a single join query.

There are performance implications to choosing which method of loading to use. Eager loading results in
one round trip to the database, but on occasion may result in complex join statements that are slow to process.
However, lazy loading results in several round trips to the database. Here eager loading is used since the join
statement will be relatively simple and we want to load the related categories in order to search over them.
The products variable is filtered using the Where operator to match products when the Name property of
the product’s Category property matches the category parameter passed into the method. This may seem
a little like overkill and you may be wondering why I didn’t just use the CategoryID property and pass in
a number rather than a string. The answer to this lies in the fact that using a category name is much more
meaningful in a URL when using routing.

This is an excellent example of why navigational properties are so useful and powerful. By using a
navigational property in my Product class, I am able to search two related entities using minimal code. If I wanted
to match products by category name, but did not use navigational properties, I would have to enter the realm of
loading category entities via the ProductsController which by convention is only meant to manage products.



■ Caution A common error often made by programmers new to using Entity Framework is using ToList()
in the wrong place. During a method, LINQ is often used for building queries and that is precisely what is does;
it simply builds up a query, it does not execute the query! The query is only executed when ToList() is called.
Novice programmers often use ToList() at the beginning of their method. The consequences of this are that
more records (usually all) will be retrieved from the database than are required, often with an adverse effect on
performance. All these records are then held in memory and processed as an in-memory list, which is usually
undesirable and can slow the web site down dramatically. Alternatively, do not even call ToList() and the
query will only be executed when the view loads. This topic is known as deferred execution due to the fact that
the execution of the query is deferred until after ToList() is called.


To finish the functionality for filtering products by category, we need to change the list of categories in
the Categories Index page into a list of hyperlinks that target the ProductsController Index method.
In order to change the categories to hyperlinks, modify the Views\Categories\Index.cshtml file by
updating the line of code

```
@Html.DisplayFor(modelItem => item.Name)
```
to:

```
@Html.ActionLink(item.Name, "Index", "Products", new { category = item.Name }, null)
```

This code uses the HTML ActionLink helper to generate a hyperlink with the link text being the
category’s name targeting the Index method of the ProductsController . The fourth parameter is
routeValue and if category is set as an expected route value, its value will be set to the category name;
otherwise, the string category=categoryname will be appended to the URL’s query string in the same
manner as we entered manually to demonstrate the product filtering was working.



## 3


### Updating the Controller for Product Searching

To add product search, modify the Index method of the Controllers\ProductsController.cs file as
shown here:

```
public ActionResult Index(string category , string search )
{
  var products = db.Products.Include(p => p.Category);
  if (!String.IsNullOrEmpty(category))
  {
    products = products.Where(p => p.Category.Name == category);
  }

  if (!String.IsNullOrEmpty(search))
  {
    products = products.Where(p => p.Name.Contains(search) ||
    p.Description.Contains(search) ||
    p.Category.Name.Contains(search));
  }
  return View(products.ToList());
}
```

First, a search parameter is added to the method and then if search is not null or empty, the products
query is modified to filter on the value of search using this code:

```
if (!String.IsNullOrEmpty(search))
{
products = products.Where(p => p.Name.Contains(search) ||
p.Description.Contains(search) ||
p.Category.Name.Contains(search));
}
```

Translated into plain English, this code says “find the products where either the product name field
contains search, the product description contains search, or the product's category name contains search”.
The code again makes use of a lambda expression but this expression is more complex and uses the logical
OR operator || . Note that there is still only one operator required on the left of the => lambda operator
despite there being multiple alternatives in the code statement to the right of => . When the query is run
against the database, the Contains method is translated to SQL LIKE and is case-insensitive.



### Adding a Search Box to the Main Site Navigation Bar

```
@using (Html.BeginForm("Index", "Products", FormMethod.Get, new { @class =
"navbar-form navbar-left" }))
{
  <div class="form-group">
  @Html.TextBox("Search", null, new { @class = "form-control", @placeholder
  = "Search Products" })
  </div>
  <button type="submit" class="btn btn-default">Submit</button>
}
```

Here an HTML form has been created to target the Index method of the ProductsController and
everything is styled using Bootstrap. The form is similar to those we have already seen in the various Create
and Edit views, but this form uses an overloaded version of the BeginForm HTML helper to specify to
use GET rather than POST when submitting the form:

```
@using (Html.BeginForm("Index", "Products",
FormMethod.Get , new { @class = "navbar-form navbar-left" }))
```

The form uses GET rather than POST so that the search term can be seen in the URL. Therefore, users
can copy it and share it with others by using other means such as e-mail or social media. By convention, GET
requests are used when making database queries that do not alter the data in the database.
One additional thing covered in this code but not seen before is the use of an HTML5 placeholder
attribute, which is used to display the text "Search Products" in the search box to indicate what it is used
for when the page is first loaded. This is done by passing an additional item into the htmlAttributes
parameter object in the line of code:

```
@Html.TextBox("Search", null, new { @class = "form-control" ,
@placeholder = "Search Products" })
```
 The name of the textbox is specified as Search and this is mapped
to the Search parameter by the MVC Framework.






### Updating the ProductsController Index Method to Filter by Category

Modify the Index method of the Controllers\ProductsController.cs file as follows in order to store
the current search in the ViewBag and generate a list of distinct categories to be stored in the ViewBag as a
SelectList .

```
public ActionResult Index(string category, string search)
{
  var products = db.Products.Include(p => p.Category);
  if (!String.IsNullOrEmpty(category))
  {
    products = products.Where(p => p.Category.Name == category);
  }
  if (!String.IsNullOrEmpty(search))
  {
    products = products.Where(p => p.Name.Contains(search) ||
    p.Description.Contains(search) ||
    p.Category.Name.Contains(search));
    ViewBag.Search = search;
  }
  var categories = products.OrderBy(p => p.Category.Name).Select(p =>
  p.Category.Name).Distinct();
  ViewBag.Category = new SelectList(categories);
  return View(products.ToList());
}
```

The search is stored in the ViewBag to allow it to be reused when a user clicks on the category filter. If it
weren’t stored, the search term would be discarded and the products would not be filtered correctly.

The code

```
var categories = products.OrderBy(p => p.Category.Name).Select(p => p.Category.
                                                                Name).Distinct();
```
 then generates a distinct list of categories ordered alphabetically. The list of categories is
not exhaustive; it only contains categories from the products that have been filtered by the search.
Finally, a new SelectList is created from the categories variable and stored in the ViewBag ready for
use in the view.


### Adding the Filter to the Products Index Page

To make the HTML page generated by the \Products\Index.cshtml file filter by category, we need a new
HTML form to submit the category to filter by. Add a new form with an HTML select control by adding the
following code to the Views\Products\Index.cshtml file after the CreateNew link.

```
<p>
  @Html.ActionLink("Create New", "Create")
  @using (Html.BeginForm("Index", "Products", FormMethod.Get))
  {
    <label>Filter by category:</label> @Html.DropDownList("Category", "All")
    <input type="submit" value="Filter" />
    <input type="hidden" name="Search" id="Search" value="@ViewBag.Search" />
  }
</p>
```

This code adds a form that targets the Index method of ProductsController using GET so that the query
string contains the values it submits. A select control is generated for the ViewBag.Category property using
the code

```
 @Html.DropDownList("Category", "All")
```
   where the "All" argument specifies the default value
for the select control. A Submit button is added to allow the user to submit the form and perform filtering.
A hidden HTML element is also included to hold the value of the current search term. It’s resubmitted so
that the search term originally entered by the user is preserved when the products are filtered by category.




### Build your queries in the correct order

 When we select a category, the rest dissapear from the dropdown list. In order to rectify
this issue, modify the Index method of the \Controllers\ProductsController.cs file as follows, so that
products are filtered by category **after the categories variable has been populated**:

```
public ActionResult Index(string category, string search)
{
  var products = db.Products.Include(p => p.Category);

  if (!String.IsNullOrEmpty(search))
  {
    products = products.Where(p => p.Name.Contains(search) ||
    p.Description.Contains(search) ||
    p.Category.Name.Contains(search));
    ViewBag.Search = search;
  }

  var categories = products.OrderBy(p => p.Category.Name).Select(p =>
                                          p.Category.Name).Distinct();

  if (!String.IsNullOrEmpty(category))
  {
    products = products.Where(p => p.Category.Name == category);
  }

  ViewBag.Category = new SelectList(categories);

  return View(products.ToList());
}
```
