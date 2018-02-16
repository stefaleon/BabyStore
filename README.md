# BabyStore

### A .NET MVC app, based on [ASP.NET MVC with Entity Framework and CSS](https://www.apress.com/gp/book/9781484221365) by Lee Naylor.


&nbsp;
## 00 Start project

* New ASP.NET Web Application.
* MVC, Individual User Accounts.


&nbsp;
## 01 Product, Category

* Add the *Product* and *Category* classes.


&nbsp;
## 02 Adding a Database Context

* Add the *DAL* folder and the *StoreContext* class.

* Add a connectionString.


&nbsp;
## 03 Adding Controllers and Views

* Add the *Categories* MVC 5 Controller with views.

&nbsp;
## 04 Adding Controllers and Views

* Add the *Products* MVC 5 Controller with views.


&nbsp;
## 05 Splitting DataAnnotations into Another File Using MetaDataType

*  Make the *Product* and *Category* classes partial, create the *ProductMetaData* and *CategoryMetaData* classes and add annotations.


&nbsp;
## 06 Filtering Products by Category

* Update the Index method in the *ProductsController* so that it receives a parameter
representing a chosen category and returns a list of products that belong to that
category.

* Transform the list shown in the *Category Index Page* to a list of hyperlinks that target
the *ProductsController* Index method rather than a list of text items.


&nbsp;
## 07 Product search

* Modify the Index method of the Controllers\ProductsController.cs.

* Add a Search Box to the Main Site Navigation Bar.


&nbsp;
## 08 Filter the Search Results by Category Using ViewBag

* Update the *ProductsController* *Index* method to filter by *Category*.

* Add the filter to the products *Index* page.



&nbsp;
## 09 Use a ViewModel

* Add the *ProductIndexViewModel* and the *CategoryWithCount* class.

* Update the *ProductsController* *Index* method to use the ViewModel.



&nbsp;
## 10 Deleting an Entity Used as a Foreign Key

* Update the *HttpPost* version of the *Delete* method in the file *\Controllers\CategoriesController.cs*.



&nbsp;
## 11 Code First Migrations

* Enable migrations for the *StoreContext*.

* Seed the Database with Test Data.



&nbsp;
## 12 Adding Data Validation and Formatting Constraints to Model Classes

* Add validation and constraints using annotations on the models.



&nbsp;
## 13 Sorting Products by Price

* Add sorting by product price.



&nbsp;
## 14 Paging

* Add pagination with the *PagedList* package.


&nbsp;
## 15 Routing

* Generate routes for categories and paging.



&nbsp;
## 16 Creating Entities to Store Image Filenames

* Add the *ProductImage* class and create the *ProductImages* table by adding a DbSet and running a migration.
