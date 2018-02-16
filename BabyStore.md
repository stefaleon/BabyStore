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

 When we select a category, the rest disappear from the dropdown list. In order to rectify
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


### Using a View Model for More Complex Filtering

Using the ViewBag to pass data to views works, but once you need to send more and more data, it becomes
messy. This is particularly true when coding because due to the dynamic nature of ViewBag , Visual Studio
offers no IntelliSense to tell you what properties are available in ViewBag . This can easily lead to coding
errors with incorrect names being used.

Rather than using ViewBag , it is better practice to use a view model for the purpose of passing
information from a controller to a view. The view is then based on this model rather than being based on
a domain model (so far, all of our views have been based on domain models). Some developers take this
concept further and base all their views solely on view models. In this book, we'll use a mixture of the view
models and domain models.

In this example, I'll show you how to add a count to the category filter control to show how many
matching products are in each category. To do this, we require a view model to hold all the information we
want to pass to the view.

### Creating a View Model

Create a new folder named ViewModels under the BabyStore project and add a new class to it named
ProductIndexViewModel .

```
using BabyStore.Models;
using System.Collections.Generic;
using System.Linq;
using System.Web.Mvc;

namespace BabyStore.ViewModels
{
    public class ProductIndexViewModel
    {
        public IQueryable<Product> Products { get; set; }
        public string Search { get; set; }
        public IEnumerable<CategoryWithCount> CatsWithCount { get; set; }
        public string Category { get; set; }
        public IEnumerable<SelectListItem> CatFilterItems
        {
            get
            {
                var allCats = CatsWithCount.Select(cc => new SelectListItem
                {
                    Value = cc.CategoryName,
                    Text = cc.CatNameWithCount
                });
                return allCats;
            }
        }
    }
    public class CategoryWithCount
    {
        public int ProductCount { get; set; }
        public string CategoryName { get; set; }
        public string CatNameWithCount
        {
            get
            {
                return CategoryName + " (" + ProductCount.ToString() + ")";
            }
        }
    }
}
```
This class file looks more complex than the code we have used so far so I will break it down step-by-step
to explain what each property is used for.

First, the file contains two classes, called ProductIndexViewModel and CategoryWithCount .
CategoryWithCount is a simple class used to hold a category name and the number of products within that
category.

The ProductCount property holds the number of matching products in a category and CategoryName
simply holds the name of the category. The CatNameWithCount property then returns both of these
properties combined into a string. An example of this property is Clothes(2) .

ProductIndexViewModel needs to hold a combination of information that was previously passed to the
view using ViewBag and also the model IEnumerable<BabyStore.Models.Product> (since this is the model
currently specified at the top of the /Views/Products/Index.cshtml file).

The first property in the class is public ```IQueryable<Product> Products { get; set; }``` . This will be
used instead of the model currently used in the view.

The second property, called ```public string Search { get; set; }``` , will replace ViewBag.Search
currently set in the ProductsController class.

The third property, called ```public IEnumerable<CategoryWithCount> CatsWithCount { get; set; }``` ,
will hold all of the CategoryWithCount items to be used inside the select control in the view.
The fourth property, Category , will be used as the name of the select control in the view.

Finally, the property ```public IEnumerable<SelectListItem> CatFilterItems``` is used to return a list of
the type SelectListItem , which will generate a value of the categoryName to be used as the value when the
HTML form is submitted and the text displayed in the format of CatNameWithCount .

### Updating the ProductsController Index Method to Use the View Model

Update the Index method of the \Controllers\ProductsController.cs file so that it matches the following
code. The changes — which use the view model rather than the ViewBag and return categories along with a
count of items — are highlighted in bold. First of all, ensure that you add a using statement to the top of the
file so that the class can access the ProductIndexViewModel class you just created.

```
        // GET: Products
        public ActionResult Index(string category, string search)
        {
            //instantiate a new view model
            var viewModel = new ProductIndexViewModel();

            //select the products
            var products = db.Products.Include(p => p.Category);

            //perform the search and save the search string to the viewModel
            if (!String.IsNullOrEmpty(search))
            {
                products = products.Where(p => p.Name.Contains(search) ||
                p.Description.Contains(search) ||
                p.Category.Name.Contains(search));
                viewModel.Search = search;
            }

            //group search results into categories and count how many items in each category
            viewModel.CatsWithCount = from matchingProducts in products
                                      where
                                      matchingProducts.CategoryID != null
                                      group matchingProducts by
                                      matchingProducts.Category.Name into
                                      catGroup
                                      select new CategoryWithCount()
                                      {
                                          CategoryName = catGroup.Key,
                                          ProductCount = catGroup.Count()
                                      };


            if (!String.IsNullOrEmpty(category))
            {
                products = products.Where(p => p.Category.Name == category);
            }

            viewModel.Products = products;

            return View(viewModel);            
        }
```

The first code change ensures that a new view model is created for use within the method:

```
ProductIndexViewModel viewModel = new ProductIndexViewModel();
```

The code ```viewModel.Search = search;``` assigns the search variable to the viewModel instead of to
ViewBag .

The third code change is a LINQ statement that populates the CatsWithCount property of viewModel
with a list of CategoryWithCount objects. In this example, I used a different form of LINQ than what I used
previously due to the complexity of the query. I used a form of LINQ known as **query syntax** to make the
query easier to read.

The statement works by grouping products by category name, where the category ID is not null,
using this code:

```
from matchingProducts in products
where
matchingProducts.CategoryID != null
group matchingProducts by matchingProducts.Category.Name into
catGroup
```

For each group, the category name and the number of products are then assigned to a
CategoryWithCount object:

```
select new CategoryWithCount()
{
  CategoryName = catGroup.Key,
  ProductCount = catGroup.Count()
};
```

The final code change assigns the products variable to the Products property of the viewModel instead
of passing it to the view and then instead passes the viewModel to the view as follows:

```
viewModel.Products = products;
return View(viewModel);
```

Note that any products not belonging to a category will not be shown in the category filter; however,
they can still be searched for.


### Modifying the View to Display the New Filter Using the View Model

Next update the \Views\Products\Index.cshtml file so that it uses the new view model to update the way it
generates the filter control, retrieves the search string, displays the table headings, and displays the list
of products. Make the following changes:

The code changes made to this file are simple but significant. The first change, ```@model BabyStore.
ViewModels.ProductIndexViewModel``` , simply tells the view to use ProductIndexViewModel as the model on
which to base the view. Note that this is now a single class and not an enumeration.

The second change, ```@Html.DropDownListFor(vm => vm.Category, Model.CatFilterItems, "All");``` ,
generates a filter control based on the CatFilterItems property of the view model as per the second parameter.

The first parameter, ```vm => vm.Category``` , specifies the HTML name of the control and hence what it will appear
as in the query string section of the URL when the form is submitted. Since the name of the control is category ,
our previous code that looks for the category parameter in the URL will continue to work correctly.

The third change ensures that the hidden search control now references the view model instead of the
ViewBag :

```
<input type="hidden" name="Search" id="Search" value="@Model.Search"/>
```

We then need to generate the table headings. This is not as straightforward as it appears because the
DisplayNameFor HTML helper method does not work with collections and we want to display headings
based on the Products property of the view model. The category heading is straightforward since this is
a property of the view model, but to display, for example, the name of the description property of the
product class, we cannot now use code such as ```@Html.DisplayNameFor(model => model.Products.
Description)``` . Instead, we need to force the helper to use an actual product entity from the products
collection by using the First() method as follows:

```
@Html.DisplayNameFor(model => model.Products.First().Description)
```

This code will now generate the table heading based on the description property of the product class.
This code will continue to work even when there are no products in the database.

■ Tip When using the DisplayNameFor HTML helper method with a collection rather than a single object,
use the first() method to enable access to the properties you want to display the name for.

The final change to the code, ```@foreach (var item in Model.Products) { ```, ensures that we now use
the Products property of the view model to display products.

Start the web site without debugging and click on View All Our Products. The Filter by Category control
now appears with a count in it, which reflects the number of matching products in each category.



## 4


### Deleting an Entity Used as a Foreign Key

So far we have updated our web site to add some useful search and filter features, and we've been able to
add some categories and products to the site. Next we'll consider what happens if we try to delete entities.

First of all, add a new category named Test Category to the web site and then add a new product
named Test Product with any description and price and assign it to Test Category . Now try to delete Test
Category by using the Categories/Index page and confirming the deletion. The web site will throw an error.

This error occurs because the database column CategoryID is used as a foreign key in the Products
table and currently there is no modification of this table when a category is deleted. This means that a
product will be left with a foreign key field that contains an ID that no longer refers to a record in the
Category table; this causes the error.

To fix this issue, the code created by the scaffolding process needs to be updated so that it sets the
foreign key of all the affected products to null . Update the HttpPost version of the Delete method in the file
\Controllers\CategoriesController.cs with the following changes highlighted in bold:

```
    // POST: Categories/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public ActionResult DeleteConfirmed(int id)
    {
        Category category = db.Categories.Find(id);
        foreach (var p in category.Products)
        {
            p.CategoryID = null;
        }
        db.Categories.Remove(category);
        db.SaveChanges();
        return RedirectToAction("Index");
    }
```

This code adds a simple foreach loop using the products navigational property of the category entity
to set the CategoryID of each product to null . When you now try to delete Test Category , it will be deleted
without an error and the CategoryID column of Test Product will be set to null in the database.



### Enabling Code First Migrations and Seeding the Database with Data

At present we have been entering data manually into the web site to create products and categories. This is
fine for testing a new piece of functionality in a development environment, but what if you want to reliably
and easily recreate the same data in other environments? This is where the feature of Entity Framework,
known as seeding, comes into play. Seeding is used to programmatically create entries in the database and
control the circumstances under which they are entered.

I am going to show you how to seed the database using a feature known as Code First Migrations .
Migrations are a way of updating the database schema based on code changes made to model classes.
We will use migrations throughout the book from now on to update the database schema.
The first thing we’re going to update is the database connection string in the web.config file so
that a new database is used for testing that the seed data works correctly.

Update the StoreContext connectionString property as follows to create a new database named BabyStore2.mdf :
```
<add name="StoreContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFile
name=|DataDirectory|\ BabyStore2 .mdf;Initial Catalog= BabyStore2 ;Integrated Security=True"
providerName="System.Data.SqlClient" />
```
Although the connectionString is shown over multiple lines in the book, be sure to keep it on a single
line in Visual Studio.


### Enabling Code First Migrations

Open the Package Manager Console by choosing View ➤ Other Windows in the main menu. This is where
all the commands to use migrations are entered. The first thing to do when using migrations is to enable
them for the database context you want to update the database schema for. If there is only one context, then
the context is optional.
In this chapter, we are interested in the product and category data, so in Package Manager Console,
enter the command:

```
Enable-Migrations -ContextTypeName BabyStore.DAL.StoreContext
```


Next add an initial migration called InitialDatabase by entering the following command in Package
Manager Console:

```
add-migration InitialDatabase
```

This command creates a new file in the Migrations folder with a name in the format
<TIMESTAMP>_ InitialDatabase.cs , where <TIMESTAMP> represents the time the file was created. The Up
method creates the database tables and the Down method deletes them. Following is the code generated in
this new class file. You can see that the Up method contains code to recreate the Categories and Products
tables along with the data types and keys.

### Seeding the Database with Test Data

When using Code First Migrations, the Seed method adds test data into a database. Generally, the data
is only added when the database is created or when some new data is added to the method. Data is not
dropped when the data model changes. When migrating to production, you will need to decide if any data is
initially required, rather than using test data, and update the seed method appropriately.
To add some new data for Categories and Products to the database, update the Seed method of the
Migrations\Configurations.cs file.

```
...code
```

This code creates a list of category and product objects and saves them to the database. To explain how
this works, we will break down the code used for the categories. First a variable named categories is created
and a list of category objects is created and assigned to it using the following code:

```
var categories = new List<Category>
{
new Category { Name="Clothes" },
new Category { Name="Play and Toys" },
new Category { Name="Feeding" },
new Category { Name="Medicine" },
new Category { Name="Travel" },
new Category { Name="Sleeping" }
};
```

The next line of code, ```categories.ForEach(c => context.Categories.AddOrUpdate(p => p.Name, c));``` ,
will add or update a category if there is not one with the same name already in the database. For this
example, we made the assumption that the category name will be unique.

The final piece of code — ```context.SaveChanges();``` —is called to save the changes to the database.

WE call this twice in the file but this is not required; you only need to call it once. However, calling it after saving
each entity type allows you to locate the source of the problem if there is an issue writing to the database.

If you do encounter a scenario where you want to add more than one entity with very similar data
(for example, two categories with the same name), you can add to the context individually as follows:

```
context.Categories.Add(new Category { Name = "Clothes" });
context.SaveChanges();
context.Categories.Add(new Category { Name = "Clothes" });
context.SaveChanges();
```

Again there is no need to save the changes multiple times, but doing so will help you to track down the
source of any error. The code used to add products follows the same pattern as that to add categories, apart
from the fact that the category entity is used to generate the CategoryID field using the following code to find a
category's ID value based on its name: ```CategoryID=categories.Single( c => c.Name == "Clothes").ID```.

### Creating the Database Using the Initial Database Migration

Now we are ready to create the new database with test data from the Seed method. In Package Manager
Console, run the command: ```update-database``` . If this works correctly, you should be informed that the
migrations have been applied and that the Seed method has run.



### Adding Data Validation and Formatting Constraints to Model Classes

At the moment, the data in the site is not validated on entry or displayed in the relevant formats such as
currency. As an example try creating a new category called 23 . You are able to do so. A user can also enter
a completely blank category name, which causes the application to throw an error when rendering the
category index.cshtml page.

Delete the new 23 category you just created. If you did create a category with a blank name, then delete
it from the database using SQL Server Object Explorer.

In Chapter 2 , I showed you how to use a MetaDataType class to add DataAnnotations to an existing class
rather than directly adding them to the class itself. Throughout the rest of the book, we are going to revert to
modifying the class itself for the sake of simplicity.


### Adding Validation and Formatting to the Category Class

We’re going to add some validation and formatting to the categories as follows:
* The name field cannot be blank
* The name field only accepts letters
* The name must be between three and fifty characters in length
In order to achieve this, you must modify the Models\Category.cs file.


```
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace BabyStore.Models
{
    public partial class Category
    {
        public int ID { get; set; }

        [Required]
        [StringLength(50, MinimumLength = 3)]
        [RegularExpression(@"^[A-Z]+[a-zA-Z''-'\s]*$")]
        [Display(Name = "Category Name")]
        public string Name { get; set; }

        public virtual ICollection<Product> Products { get; set; }
    }
}
```

The [Required] attribute marks the property as being required, i.e., it cannot be null or empty, while
the [StringLength(50, MinimumLength = 3)] attribute specifies that the string entered into the field must
be between 3 and 50 characters in length.

The final attribute— ```[RegularExpression(@"^[A-Z]+[a-zA-Z''-
'\s]*$")]``` —uses a regular expression to specify that the field must only contain letters and spaces and start
with an uppercase letter. In simple terms, this expression says the first character must be an uppercase letter
followed by a repetition of letter and spaces. I don't cover regular expressions in this book since several
tutorials are available online. If you want to translate a regular expression into something more meaningful
or view a library of commonly used expressions, try using the site https://regex101.com/ .


Next, start the web site without debugging and click on Shop by Category. Figure 4-8 shows the resulting
page. Instead of displaying the Categories index page, the web site shows an error message informing you
that the model backing StoreContext has changed.


This issue is caused because the Category class now has changes in it that need to be applied to the
database. Two of the three attributes we added need to be applied to the database to change the rules for
the Name column. This will ensure that the column cannot be null and also apply a maxLength attribute. The
regular expression and the minimum length are not applicable to the database.

In order to fix this issue, open Package Manager Console and add a new migration by running the

```
add-migration CategoryNameValidation
```
 command . A new migration file will be created.

 To apply these changes to the database, run the ```update-database``` command in Package Manager
Console. The Name column of the Categories table in the database will now be updated.


Now run the web site again and navigate to the Category Create page by clicking on Shop by Category
on the home page. Then click Create New on the Categories index page. Try to create a blank category; the
web site will inform you that this is not allowed.


Now try to create a category named Clothes 2 .
As you can see, the message  is not exactly user friendly. Fortunately, ASP.NET
MVC allows us to override the error message text by entering an extra parameter into the attribute. To add
some more user friendly error messages for each field, update the \Models\Category.cs file as follows:

```
[Required(ErrorMessage = "The category name cannot be blank")]
[StringLength(50, MinimumLength = 3, ErrorMessage = "Please
 enter a category name between 3 and 50 characters in length" )]
[RegularExpression(@"^[A-Z]+[a-zA-Z''-'\s]*$", ErrorMessage = "Please
enter a category name beginning with a capital letter and made up
of letters and spaces only" )]
```

■ Note This error message should be entered on a single line in Visual Studio. They are split here simply for
book formatting. Alternatively, if you want to split them over two lines, you must use closing and opening quote
with a plus sign: " + " .


Now start the web site without debugging and try creating the Clothes 2 category again. This time, the
more meaningful error message is displayed.


### Adding Formatting and Validation to the Product Class

The product class contains properties that require more complex attributes such as formatting as currency
and displaying a field over multiple lines. Update the Models\Product.cs file with the following code.

```
using System.ComponentModel.DataAnnotations;

namespace BabyStore.Models
{
    public partial class Product
    {
        public int ID { get; set; }

        [Required(ErrorMessage = "The product name cannot be blank")]
        [StringLength(50, MinimumLength = 3, ErrorMessage = "Please enter a product name between 3 and 50 characters in length")]
        [RegularExpression(@"^[a-zA-Z0-9'-'\s]*$", ErrorMessage = "Please enter a product name made up of letters and numbers only")]
        public string Name { get; set; }

        [Required(ErrorMessage = "The product description cannot be blank")]
        [StringLength(200, MinimumLength = 10, ErrorMessage = "Please enter a product description between 10 and 200 characters in length")]
        [RegularExpression(@"^[,;a-zA-Z0-9'-'\s]*$", ErrorMessage = "Please enter a product description made up of letters and numbers only")]
        [DataType(DataType.MultilineText)]
        public string Description { get; set; }

        [Required(ErrorMessage = "The price cannot be blank")]
        [Range(0.10, 10000, ErrorMessage = "Please enter a price between 0.10 and 10000.00")]
        [DataType(DataType.Currency)]
        [DisplayFormat(DataFormatString = "{0:c}")]
        public decimal Price { get; set; }

        public int? CategoryID { get; set; }

        public virtual Category Category { get; set; }
    }
}
```

There are some new entries here that we have not seen before. The code ```[DataType(DataType.
MultilineText)]``` tells the UI to display the input element for the description field as a text area when used
in the edit and create views.

```
[DataType(DataType.Currency)]
```
is used to give a hint to the UI as to what the format should be and it
emits HTML5 date attributes to be used by HTML 5 browsers to format input elements. At the moment, web
browser implementation of these attributes is unfortunately patchy.

```
[DisplayFormat(DataFormatString = "{0:c}")]
```
 specifies that the price property should be
displayed in currency format, i.e., £1,234.56 (with the currency set by the server locale). Generally either of
these attributes should work and display the price formatted as currency. We have included them both here
for completeness.

Now build the solution and then, in Package Manager Consoler, run the command

```
add-migration ProductValidation
```
 followed by ```update-database``` to add the range and nullable settings to the database.

Start the web site without debugging and click on View All Our Products. The list of
products shows with the price field now formatted as currency due to the data annotations we made.

■ Note It is possible to use the currency format when editing by using the code [DisplayFormat(Data
FormatString = "{0:c}", ApplyFormatInEditMode = true)] , but I don't recommend that you do this,
because the price when editing will display in the format £9,999.99. When you then try to submit the edit form,
the price will fail validation because £ is not a number.


Click on the Details link and you will see that the price is also now formatted as currency. To see the full
effect of the changes to the product class, we need to try creating and editing a product.



This is a big improvement on our default view and the validation applied to it; however, there are still
some issues. The price input still displays a default error message when a number is not entered, which can
be fixed in a couple of ways.

One way to fix this is to overwrite the data-val-number HTML attribute by modifying the EditorFor
code for the price field. You do this by passing in a new HTML attribute, as follows:

```
@Html.EditorFor(model => model.Price, new { htmlAttributes = new { @class = "form-control",
data_val_number = "The price must be a number." } })
```

**This change must implemented in each view that allows you to edit price, making it more difficult
to maintain**.

An **alternative and more maintainable way** to remedy this is to use a regular expression in the product
class in the same manner as before, by updating the price property as follows:

```
//...
[RegularExpression("[0-9]+(\\.[0-9][0-9]?)?",
ErrorMessage = "The price must be a number up to two decimal places")]
public decimal Price { get; set; }
```

This regular expression allows a number optionally followed by a decimal point, plus another one
or two numbers. It allows numbers of the following format—1, 1.1, and 1.10—but not 1. without anything
following the decimal point.



## 5

## Sorting, Paging, and Routing


### Sorting Products by Price

To demonstrate sorting, I’ll show you a simple example to sort products by price, allowing users to order
products by price.
First of all, add a new switch statement to the Index method of the Controllers\ProductsController.cs
file, as highlighted in the following code, so that the products are reordered by price:

```
// GET: Products
public ActionResult Index(string category, string search , string sortBy )
{
  //...
  //sort the results
  switch (sortBy)
  {
      case "price_lowest":
          products = products.OrderBy(p => p.Price);
          break;
      case "price_highest":
          products = products.OrderByDescending(p => p.Price);
          break;
      default:
          break;
  }

  viewModel.Products = products;

  return View(viewModel);            
}
```

This new code uses the Entity Framework OrderBy and OrderByDescending methods to sort
products by ascending and descending price. Run the application without debugging and manually
change the URL to test that sorting works as expected, by using the Products?sortBy=price_lowest and
Products?sortBy=price_highest URLs. The products should reorder with the lowest priced item at the
top and the highest priced item at the top, respectively.


### Adding Sorting to the Products Index View

We now need to add some user interface controls for sorting into the web site to allow users to choose
how they want to sort. To demonstrate this, add a select list and populate it with values and text from a
dictionary type.

First of all, add the following highlighted SortBy and Sorts properties to the ProductIndexViewModel
class in the \ViewModels\ProductIndexViewModel.cs file:


```
public class ProductIndexViewModel
{
    //...
    public string SortBy { get; set; }
    public Dictionary<string, string> Sorts { get; set; }

    //...
}
```


The SortBy property will be used as the name of the select element in the view and the Sorts property
will be used to hold the data to populate the select element.

Now we need to populate the Sorts property from the ProductController class. Modify the
\Controllers\ProductsController.cs file to add the following line of code to the end of the Index method
prior to returning the View:

```
// GET: Products
public ActionResult Index(string category, string search, string sortBy)
{
    //...
    viewModel.Products = products;

    viewModel.Sorts = new Dictionary<string, string>
    {
        {"Price low to high", "price_lowest" },
        {"Price high to low", "price_highest" }
    };

    return View(viewModel);     
}
```



Finally, we need to add the control to the view so that users can make a selection. To achieve this, add
the highlighted code to the Views\Products\Index.cshtml file after the filter by category code as follows:

```
@model BabyStore.ViewModels.ProductIndexViewModel

//...
<p>
    @Html.ActionLink("Create New", "Create")
    @using (Html.BeginForm("Index", "Products", FormMethod.Get))
    {
        <label>Filter by category:</label>
        @Html.DropDownListFor(vm => vm.Category, Model.CatFilterItems, "All");
          <label>Sort by:</label>
          @Html.DropDownListFor(vm => vm.SortBy, new SelectList(Model.Sorts, "Value", "Key"),
          "Default")
        <input type="submit" value="Filter" />
        <input type="hidden" name="Search" id="Search" value="@Model.Search" />
    }
</p>
//...
```

This new select control uses the SortBy property from the view model as its name. It populates itself
with the data from the view model’s Sorts property using the second entry in each line of the dictionary as
the value submitted by the control (specified by "Value" ) and the first entry in each line as the text displayed
to the user (specified by "Key" ).




### Adding Paging

In this section, I will show you a way to add paging to allow users to page through the product search
results rather than showing them all in one large list. This code will use the popular NuGet package
PagedList.Mvc , which is written and maintained by Troy Goode. I have chosen to use this as an
introduction to paging because it is easy to set up and use. Later in the book, I will show you how to write
your own asynchronous paging code and an HTML helper to display paging controls.


### Installing PagedList.Mvc

First of all, we need to install the package. Open the Project menu and then choose Manage NuGet
Packages in order to display the NuGet Package Manager window. In this window, select the browse option
and then search for pagedlist . Then install the latest version of PagedList.Mvc
by clicking on the Install link. When you install PagedList.Mvc , the PagedList package is
also installed.


### Updating the View Model and Controller for Paging
Once PagedList.Mvc is installed, the first thing that needs to be modified is ProductIndexViewModel , so that
the Products property is changed to the type IPagedList . Modify the ViewModels\ProductIndexViewModel.cs file
to update the code highlighted here:

```
using PagedList;
//...

namespace BabyStore.ViewModels
{
    public class ProductIndexViewModel
    {
        public IPagedList<Product> Products { get; set; }        
        public string Search { get; set; }  
        //...
```


We now need to modify the Index method of the ProductsController class so that it returns Products
as a PagedList (achieved by using the ToPagedList() method). A default sort order also needs to be set in
order to use PagedList . First of all, add the code using PagedList; to the using statements at the top of
the file. Then modify the Controllers\ProductsController.cs file, as highlighted, in order to use the new
PagedList package.

```
public ActionResult Index(string category, string search, string sortBy, int? page )
{
//...
    if (!String.IsNullOrEmpty(category))
    {
        products = products.Where(p => p.Category.Name == category);
        viewModel.Category = category;
    }

    //sort the results
    switch (sortBy)
    {
        //...
        default:
            products = products.OrderBy(p => p.Name);
            break;
    }

    const int PageItems = 3;
    int currentPage = (page ?? 1);
    viewModel.Products = products.ToPagedList(currentPage, PageItems);
    viewModel.SortBy = sortBy;
    viewModel.Sorts = new Dictionary<string, string>
    {
        {"Price low to high", "price_lowest" },
        {"Price high to low", "price_highest" }
    };

    return View(viewModel);
}
```

The first change adds the parameter int? page , which is a nullable integer and will represent the
current page chosen by the user in the view. When the Products index page is first loaded, the user will not
have selected any page, hence this parameter can be null.

We also need to ensure that the current category is saved to the view model so we have added this line
of code to ensure that you can page within a category:

```
viewModel.Category = category;
```

The code
```
products = products.OrderBy(p => p.Name);
```
is then used to set a default order of products
because PagedList requires the list it receives to be sorted.

Next, we specify the number of items to appear on each page by adding a constant using the line of code

```
const int PageItems = 3;
```

 We then declare an integer variable

 ```
 int currentPage = (page ?? 1);
 ```
 to hold the current page number and take the value of the page parameter, or 1, if the page variable is null.

The products property of the view model is then assigned a PagedList of products specifying the
current page and the number of items per page using the code

```
viewModel.Products = products.ToPagedList(currentPage, PageItems);
```

Finally, the sortBy value is now saved to the view model so that the sort order of the products list is
preserved when moving from one page to another by the code:

```
viewModel.SortBy = sortBy;
```



### Updating the Products Index View for Paging

Having implemented the paging code in our view model and controller, we now need to update the
\Views\Products\Index.cshtml file to display a paging control so that the user can move between pages.

We’ll also add an indication of how many items were found. To achieve this, modify the file to add a new
using statement, add an indication of the total number of products found, and display paging links at the
bottom of the page, as highlighted in the following code:

```
@model BabyStore.ViewModels.ProductIndexViewModel
@using PagedList.Mvc

@{
    ViewBag.Title = "Products";
}

<h2>@ViewBag.Title</h2>

<p>
    @(String.IsNullOrWhiteSpace(Model.Search) ? "Showing all" : "Your search for " +
        Model.Search + " found") @Model.Products.TotalItemCount products
</p>

//...


<div>
    Page @(Model.Products.PageCount < Model.Products.PageNumber ? 0 :
                    Model.Products.PageNumber) of @Model.Products.PageCount
    @Html.PagedListPager(Model.Products, page => Url.Action("Index",
             new
                 {
                     category = @Model.Category,
                     Search = @Model.Search,
                     sortBy = @Model.SortBy,
                     page
                 }))
</div>
```


The indication of how many products were found is displayed using the code:

```
<p>
@(String.IsNullOrWhiteSpace(Model.Search) ? "Showing all" : "Your search for " +
Model.Search + " found") @Model.Products.TotalItemCount products
</p>
```
This code uses the `?` : (also known as ternary) operator to check if the search term is null or made
up of whitespace. If this is true, the output of the code will be "Showing all xx products" or else if the
user has entered a search term, the output will be "Your search for search term found xx products" .

In effect, this operates as a shorthand if statement. More information on the ?: operator can be found at
https://msdn.microsoft.com/en-gb/library/ty67wk28.aspx .

Finally, the paging links are generated by this new code:

```
<div>
  Page @(Model.Products.PageCount < Model.Products.PageNumber ? 0 :
    Model.Products.PageNumber) of @Model.Products.PageCount
  @Html.PagedListPager(Model.Products, page => Url.Action("Index",
    new { category = @Model.Category,
          Search = @Model.Search,
          sortBy = @Model.SortBy,
          page
      }))
</div>
```

This code is wrapped in a div tag for presentation purposes. The first code line uses the `?`: operator to
decide whether or not there are any pages to display. It displays "Page 0 of 0" or "Page x of y" where x is
the current page and y the total number of pages.

The next line of code uses the HTML PagedListPager helper that comes as part of the PagedList.Mvc namespace.
This helper takes the list of products and produces a hyperlink to each page.

Url.Action is used to generate a hyperlink targeting the Index view containing the page parameter. We have added an
anonymous type to the helper method in order to pass the current category, search, and sort order to the
helper so that each page link contains these in its querystring. This means that the search term, chosen
category, and sort order are all preserved when moving from one page to another. Without them, the list of
products would be reset to show all the products.




### Routing

So far we have been using parameters in the querystring portion of the URL to pass data for categories and
paging to the Index action method in the ProductController class. These URLs follow the standard format
such as /Products?category=Sleeping&page=2 , and although functional, these URLs can be improved
upon by using the ASP.NET Routing feature. It generates URLs in a more “friendly” format that is more
meaningful to users and search engines. ASP.NET routing is not specific to MVC and can also be used with
Web Forms and Web API; however, the methods used are slightly different when working with Web Forms.


To keep things manageable, we’re going to generate routes only for categories and paging. There won’t
be a route for searching or sorting due to the fact that routing requires a route for each expected combination
that can appear and each parameter needs some way of identifying itself. For example, we use the word “page”
to prefix each page number in the routes that use it. It is possible to make routing overly complex by trying to
add a route for everything. It’s also worth noting that any values submitted by the HTML form for filtering by
category will still generate URLs in the “old” format because that is how HTML forms work by design.


One of the most important things about routing is that routes have to be added in the order of most
specific first, with more general routes further down the list. The routing system searches down the routes
until it finds anything that matches the current URL and then it stops. If there is a general route that matches
a URL and a more specific route that also matches, but it is defined below the more general route then the
more specific route will never be used.


### Adding Routes

We are going to take the same approach to adding routes as used by the scaffolding process when the project
was created and add them to the \App_Start\RouteConfig.cs file in this format:

```
routes.MapRoute(
  name: "Name",
  url: "Rule",
  defaults: DefaultValues
);
```

The name parameter represents the name of the route and can be left blank; however, we’ll be using
them in this book to differentiate between routes.
The url parameter contains a rule for matching the route to a URL format. This can contain several
formats and arguments, as follows:
* The url parameter is divided into segments, with each segment matching sections of
the URL.
* A URL has to have the same number of segments as the url parameter in order to
match it, unless either defaults or a wildcard is specified (see the following bullet
points for an explanation of each of these).
* Each segment can be:
  * A static element URL, such as “Products”. This will simply match the URL
/Products and will call the relevant controller and action method.
  * A variable element that is able to match anything. For example,
"Products/{category}" will match anything after Products in a URL and
assign it to {category} and {category} can then be used in the action method
targeted by the route.
  * A combination of static and variable elements that will match anything in the
same URL segment matching the format specified. For example, "Products/
Page{page}" will match URLs such as Products/Page2 or Products/Page99 and
assign the value of the part of the URL following Page to {page} .
  * A catch-all wildcard element, for example "Products/{`*`everything} , will
map everything following the Products section of the URL into the everything
variable segment. This is done regardless of whether it contains slashes or not.
We won’t use wildcard matches in this project.
  * Each segment can also be specified as being optional or having a default value if the
corresponding element of the URL is blank. A good example of this is the default
route specified when the project was created. This route uses the following code to
specify default values for the controller and action method to be used. It also defines
the id element as being optional:

```
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Home", action = "Index",
                          id = UrlParameter.Optional }
);
```




To start with routes, add a route for allowing URLs in the format /Products/Category (such as /
Products/Sleeping to display just the products in the Sleeping category). Add the following highlighted
code to the RegisterRoutes method in the \App_Start\RouteConfig.cs file, above the Default route:

```
namespace BabyStore
{
    public class RouteConfig
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

            routes.MapRoute(
                name: "ProductsbyCategory",
                url: "Products/{category}",
                defaults: new { controller = "Products", action = "Index" }
            );

            //...
        }
    }
}
```

Start the web site without debugging and click on Shop by Category, then click on Sleeping. The link will
now open the /Products/Sleeping URL, due to the new ProductsbyCategory route.

So far so good; everything looks like it’s working okay and you can now use the URLs in the format
Products/Category . However, there is a problem. Try clicking on the Create New link. The product create
page no longer appears and instead a blank list of categories is displayed. The reason for this is that the new
route treats everything following Products in the URL as a category. There is no category named Create, so
no products are returned.

Now click the back button to go back to the products list with some products displayed. Try clicking on
the Edit, Details, and Delete links. They still work! You may be wondering why this is; well, the answer lies
in the fact that the working links all have an ID parameter on the end of them. For example, the edit links
take the format /Products/Edit/6 and this format matches the original Default route ( "{controller}/
{action}/{id}" ) rather than the new ProductsbyCategory route ( "Products/{category}" ).

To fix this issue, we need to add a more specific route for the Products/Create URL. Add a new route to
the RegisterRoutes method in the App_Start\RouteConfig.cs file above the ProductsbyCategory route.


```
namespace BabyStore
{
    public class RouteConfig
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

            routes.MapRoute(
                name: "ProductsCreate",
                url: "Products/Create",
                defaults: new { controller = "Products", action = "Create" }
            );

            routes.MapRoute(
                name: "ProductsbyCategory",
                url: "Products/{category}",
                defaults: new { controller = "Products", action = "Index" }
            );

            //...
        }
    }
}
```

Start the web site without debugging and click on the Create Product link. It now works again
because of the ProductsCreate route. It’s very important to add the ProductsCreate route above the
ProductsByCategory route; otherwise, it will never be used. If it was below the ProductsByCategory route,
the routing system would find a match for the "Products/{category}" URL first and then stop searching for
a matching route.



Next we’re going to add a route for paging so that the web site can use URLs in the format /Products/Page2 .
Update the RegisterRoutes method of the App_Start\RouteConfig.cs file to add a new route to the file
above the ProductsbyCategory route, as follows:

```
namespace BabyStore
{
    public class RouteConfig
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

            routes.MapRoute(
                name: "ProductsCreate",
                url: "Products/Create",
                defaults: new { controller = "Products", action = "Create" }
            );

            routes.MapRoute(
                name: "ProductsbyPage",
                url: "Products/Page{page}",
                defaults: new
                { controller = "Products", action = "Index" }
            );

            routes.MapRoute(
                name: "ProductsbyCategory",
                url: "Products/{category}",
                defaults: new { controller = "Products", action = "Index" }
            );

            //...
        }
    }
}
```

This new ProductsByPage route will match any URLs with the format Products/PageX , where X is the
page number. Again this route must appear before the ProductsbyCategory route; otherwise, it will never
get used. Try the new ProductsbyPage route by starting the web site without debugging and clicking View All
Our Products. Then click on a page number in the paging control at the bottom of the page. The URL should
now appear in the format Products/PageX . For example, clicking on number 4
in the paging control, generates the URL Products/Page4 .




So far we have added routes for Products/Category and Product/PageX , but we have nothing
for Product/Category/PageX . To add a new route that allows this, add the following code above the
ProductsByPage route in the RegisterRoutes method of the App_Start\RouteConfig.cs file:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;
using System.Web.Routing;

namespace BabyStore
{
    public class RouteConfig
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

            routes.MapRoute(
                name: "ProductsCreate",
                url: "Products/Create",
                defaults: new { controller = "Products", action = "Create" }
            );

            routes.MapRoute(
                name: "ProductsbyCategorybyPage",
                url: "Products/{category}/Page{page}",
                defaults: new { controller = "Products", action = "Index" }
            );

            routes.MapRoute(
                name: "ProductsbyPage",
                url: "Products/Page{page}",
                defaults: new
                { controller = "Products", action = "Index" }
            );

            //...
        }
    }
}
```

Start the web site without debugging and click on Shop by Category and then click on Sleeping. Then
click on page 2 in the paging control. The URL now generated is Products/Sleeping/Page2 because of the
new ProductsbyCategorybyPage route.



We now appear to have added all the new routes, but there are still some issues with how the
new routes affect the site. To see the first remaining issue, start with the web site filtered as shown in
Figure 5-8 . Then try to choose another category from the drop-down and clicking the filter button. The
results are not filtered and remain on the Sleeping category. This is because the HTML form no longer
targets the ProductsController index method correctly. To resolve this issue, we need to add one final route
to the /App_Start/RouteConfig.cs file and then configure the HTML form to use it.
First of all, add a new route above the default route in the RegisterRoutes method of the
App_Start\RoutesConfig.cs file:

```
namespace BabyStore
{
    public class RouteConfig
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            //...

            routes.MapRoute(
                name: "ProductsIndex",
                url: "Products",
                defaults: new { controller = "Products", action = "Index" }
            );

            routes.MapRoute(
                name: "Default",
                url: "{controller}/{action}/{id}",
                defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
            );
        }
    }
}
```

This new route is named ProductsIndex and it creates a route that targets the Index action method of
the ProductsController class. We’ve created this to give the web site a way to target this method when using
URL links and forms.



## 6

## Managing Product Images:Many-to-Many Relationships

This chapter covers how to create a new entity to manage images, how to upload image files using HTML
forms and associate them with products using a many-to-many relationship, and how to save images to the
filesystem. This chapter also introduces more complex error handling to add custom errors to the model in
order to display them back to the user.

### Creating Entities to Store Image Filenames

For this project, we’re going to store the image files within the web project using the filesystem. The database
will contain data relating the filesystem’s name of the image with one or more products. To begin modeling
image storage, add a new class named ProductImage to the Models folder as follows:

```
using System.ComponentModel.DataAnnotations;

namespace BabyStore.Models
{
    public class ProductImage
    {
        public int ID { get; set; }

        [Display(Name = "File")]
        public string FileName { get; set; }
    }
}
```

You are probably wondering why we added an extra class that basically maps a string to a product
rather than simply adding a collection of strings to the Product class. This is a common question raised by
developers when using Entity Framework. The reason is that Entity Framework cannot model a collection of
strings in the database; it requires the collection to be modeled as we have done with the strings stored in a
distinct class.


Now update the DAL\StoreContext.cs file to add a new property for ProductImages as follows:

```
using BabyStore.Models;
using System.Data.Entity;

namespace BabyStore.DAL
{
    public class StoreContext: DbContext
    {
        public DbSet<Product> Products { get; set; }
        public DbSet<Category> Categories { get; set; }

        public DbSet<ProductImage> ProductImages { get; set; }
    }
}
```

Next, create a migration to add the new ProductImage entity as a table by entering
```
add-migration ProductImages
```
into Package Manager Console and pressing Return. Then run the `update-database`
command to update the database and create the new ProductImages table.
