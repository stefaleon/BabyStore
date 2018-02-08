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


Ch02end
