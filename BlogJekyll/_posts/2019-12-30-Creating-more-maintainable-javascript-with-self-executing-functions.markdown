---
layout: post
title:  "Creating more maintainable javascript with self-executing functions"
date:   2019-12-30 00:00:00 +0100

tags: JavaScript
---
I still mostly use regular JavaScript with jQuery in my applications. Before I started writing self-executing modules, my JavaScript was embedded all over the place in HTML and it was untestable (except for running a page in the browser). 

A year or so ago a colleague introduced me to the concept of self-executing JavaScript functions. (Thank you Gijs Stoeldraaijers!) After adopting this practice, I found that I could write JavaScript modules with keeping the SOLID principles in mind. The most important benefits are creating smaller modules with a single purpose that can be tested and dependency injection support to make unit testing even easier. Even without adopting unit testing right away the improvements to maintainability were immense for me. 

I’ll not go into JavaScript unit testing in this post and do an in-depth post as my next blog post. If you want to look into it, without waiting for that, please look at the [Microsoft Docs page on Unit testing JavaScript and TypeScript in Visual Studio]( https://docs.microsoft.com/en-us/visualstudio/javascript/unit-testing-javascript-with-visual-studio?view=vs-2019#unit-tests-in-other-project-types).

**Table of contents:**
* Table of Contents
{:toc}

### Example of the old way of working (embedding JavaScript in the page)
Below, I have created an example of my old way of writing JavaScript on an overview page that loads data from an API endpoint and has some filter capabilities.

The data templates are hardcoded strings in the JavaScript and the search button event hookup is done through HTML markup.

A working Visual Studio project with the code listed below can be found in my [JavaScriptModuleExamples repository on GitHub with the tag "V1.0-Embedded-JavaScript"](https://github.com/SamanthaNeilen/JavascriptModulesExamples/tree/V1.0-Embedded-JavaScript).

```html
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}
<div class="col">
    <h1>Customer overview</h1>

    @* Searchform *@
    <div id="search-customer">
        <div class="form-group row">
            <label for="searchCustomerName" class="col-sm-2 col-form-label">@Texts.Name</label>
            <div class="col-sm-5">
                <input type="text" class="form-control" id="searchCustomerName" value="">
            </div>
        </div>
        <div class="form-group row">
            <label for="searchEmail" class="col-sm-2 col-form-label">@Texts.Email</label>
            <div class="col-sm-5">
                <input type="text" class="form-control" id="searchEmail" value="">
            </div>
        </div>
        <div class="form-group row">
            <label for="searchCountry" class="col-sm-2 col-form-label">@Texts.Country</label>
            <div class="col-sm-5">
                <input type="text" class="form-control" id="searchCountry" value="">
            </div>
        </div>
        <button type="button" class="btn btn-sm btn-primary col-sm-2" onclick="filterData()">@Texts.Search</button>
    </div>

    @* Grid for data *@
    <table class="table mt-2">
        <thead>
            <tr>
                <th scope="col">@Texts.Name</th>
                <th scope="col">@Texts.Email</th>
                <th scope="col">@Texts.Country</th>
            </tr>
        </thead>
        <tbody id="customer-table-body">
            <tr class="loading-row">
                <td colspan="3"><progress></progress></td>
            </tr>
        </tbody>
    </table>
</div>

@section Scripts {
    <script type="text/javascript">
        const CUSTOMER_ROW_SELECTOR = '.customer-row',
            LOADING_ROW_SELECTOR = '.loading-row',
            DISPLAY_NONE_CLASS = 'display-none',
            CUSTOMER_TABLE_BODY = $('#customer-table-body');

        function loadData() {
            showLoader();

            $.get('https://localhost:44313/api/customer')
                .done(function (customerList, textStatus, jqXHR) {
                    CUSTOMER_TABLE_BODY.html(getDataRows(customerList));
                    showCustomerRows(CUSTOMER_TABLE_BODY.find(CUSTOMER_ROW_SELECTOR));
                })
                .fail(function (jqXHR, textStatus, errorThrown) {
                    alert('error ' + textStatus + ' ' + errorThrown);
                });
        }

        function getDataRows(customerList) {
            const rowTemplate = '<tr class="customer-row"><td>{name}</td><td>{email}</td><td>{country}</td></tr>';
            let dataRows = '<tr class="loading-row"><td colspan="3"><progress></progress></td></tr>';

            $.each(customerList,
                function (index, customer) {
                    dataRows += rowTemplate
                        .replace('{name}', customer.name)
                        .replace('{email}', customer.email)
                        .replace('{country}', customer.country);
                });

            return dataRows;
        }

        function showLoader() {
            CUSTOMER_TABLE_BODY.find(LOADING_ROW_SELECTOR).removeClass(DISPLAY_NONE_CLASS);

            const dataRows = CUSTOMER_TABLE_BODY.find(CUSTOMER_ROW_SELECTOR);
            $.each(dataRows,
                function (index, row) {
                    $(row).addClass(DISPLAY_NONE_CLASS);
                });
        }

        function showCustomerRows(customerRows) {
            $.each(customerRows,
                function (index, row) {
                    $(row).removeClass(DISPLAY_NONE_CLASS);
                });
            CUSTOMER_TABLE_BODY.find(LOADING_ROW_SELECTOR).addClass(DISPLAY_NONE_CLASS);
        }

        function filterData() {
            showLoader();

            const customerRows = CUSTOMER_TABLE_BODY.find(CUSTOMER_ROW_SELECTOR);
            const searchName = $('#searchCustomerName').val();
            const searchEmail = $('#searchEmail').val();
            const searchCountry = $('#searchCountry').val();

            const rowsToShow = $.grep(customerRows,
                function (customerRow) {
                    let columns = $(customerRow).find('td');

                    return isMatchForFilter(searchName, columns[0].innerHTML) &&
                        isMatchForFilter(searchEmail, columns[1].innerHTML) &&
                        isMatchForFilter(searchCountry, columns[2].innerHTML);


                });

           showCustomerRows(rowsToShow);
        }

        function isMatchForFilter(searchValue, valueInRow) {
            return searchValue === '' ||
                valueInRow.indexOf(searchValue) !== -1;
        }

        $(document).ready(function () {
            loadData();
        });
    </script>
}
```

## Self-executing functions

An example of a self-executing module is shown below. The proper term for the self-executing module is Immediately Invoked Function Expressions or IIFE.

``` javascript
(function ($, someOtherInjectedModule) {
    const CURRENT_MODULE = {};
    const PRIVATE_VARIABLE = 'Hello world!';

    CURRENT_MODULE.myPublicFunction = function() {
        privateFunction();
    };

    function privateFunction() {
        alert(PRIVATE_VARIABLE);
    }

    window.ModuleExample = CURRENT_MODULE;

})(jQuery, window.SomeOtherModule);

$(document).ready(function() {
    window.ModuleExample.myPublicFunction();
});
```

The function parameters make it possible to inject dependencies. This has 2 benefits.

1. You can inject mocks when unit testing. This makes unit testing much easier.

2. All the external dependencies, that the class relies on, are shown at the top and bottom of the file. 

Another great benefit that this pattern gives is variable scoping. All variables scoped within the module are scoped to the function and do not have to be global. 

The CURRENT_MODULE object and all its functions/properties are exposed globally by assigning it to a global window property. All the other “private” functions and variables are never exposed outside the scope of the self-executing function.

The  example as shown in the first code listing of this post rewritten with an IIFE is shown below.

A working Visual Studio project with the code listed below can be found in my [JavaScriptModuleExamples repository on GitHub](https://github.com/SamanthaNeilen/JavascriptModulesExamples).

customer-overview.js:

```javascript
(function ($, customerService, utilitiesModule) {
    const CURRENT_MODULE = {},
        CUSTOMER_ROW_SELECTOR = '.customer-row',
        LOADING_ROW_SELECTOR = '.loading-row',
        CUSTOMER_TABLE_BODY = $('#customer-table-body');

    CURRENT_MODULE.initialize = function() {
        $('#customerOverviewSearchButton').on('click', CURRENT_MODULE.filterData);
        CURRENT_MODULE.loadData();
    };

    CURRENT_MODULE.loadData = function () {
        showLoader();

        customerService.getCustomers(showCustomerData);
    };

    CURRENT_MODULE.filterData = function () {
        showLoader();

        const customerRows = CUSTOMER_TABLE_BODY.find(CUSTOMER_ROW_SELECTOR);
        const searchName = $('#searchCustomerName').val();
        const searchEmail = $('#searchEmail').val();
        const searchCountry = $('#searchCountry').val();

        const rowsToShow = $.grep(customerRows,
            function (customerRow) {
                const columns = $(customerRow).find('td');

                return utilitiesModule.isMatchForFilter(searchName, columns[0].innerHTML) &&
                    utilitiesModule.isMatchForFilter(searchEmail, columns[1].innerHTML) &&
                    utilitiesModule.isMatchForFilter(searchCountry, columns[2].innerHTML);
            });

        showCustomerRows(rowsToShow);
    };

    function showCustomerData(customerList) {
        CUSTOMER_TABLE_BODY.html(getDataRows(customerList));
        showCustomerRows();
    }

    function getDataRows(customerList) {
        const outerHtmlProperty = 'outerHTML';
        const dataRowTemplateId = 'data-row-template';
        const rowTemplate = $('#' + dataRowTemplateId).prop(outerHtmlProperty)
            .replace('id="' + dataRowTemplateId + '"', '');

        let dataRows = $(LOADING_ROW_SELECTOR).prop(outerHtmlProperty);

        $.each(customerList,
            function (index, customer) {
                dataRows += rowTemplate
                    .replace('{name}', customer.name)
                    .replace('{email}', customer.email)
                    .replace('{country}', customer.country);
            });

        return dataRows;
    }

    function showLoader() {
        utilitiesModule.showElement(LOADING_ROW_SELECTOR, CUSTOMER_TABLE_BODY);

        forEachCustomerRow(function (row) {
            utilitiesModule.hideElement(row, CUSTOMER_TABLE_BODY);
        });
    }

    function showCustomerRows(filteredRows) {
        forEachCustomerRow(function (row) {
            utilitiesModule.showElement(row, CUSTOMER_TABLE_BODY);
        }, filteredRows);

        utilitiesModule.hideElement(LOADING_ROW_SELECTOR, CUSTOMER_TABLE_BODY);
    }

    function forEachCustomerRow(callBack, filteredRows) {
        let customerRows = CUSTOMER_TABLE_BODY.find(CUSTOMER_ROW_SELECTOR);
        if (filteredRows !== undefined) {
            customerRows = filteredRows;
        }

        $.each(customerRows,
            function (index, row) {
                callBack($(row));
            });
    }

    window.customerOverviewModule = CURRENT_MODULE;

})(jQuery, window.customerService, window.utilitiesModule);

$(document).ready(function () {
    window.customerOverviewModule.initialize();
});
```
utilities.js

```javascript
(function ($) {
    const CURRENT_MODULE = {},
        DISPLAY_NONE_CLASS = 'display-none';

    CURRENT_MODULE.isMatchForFilter = function (searchValue, valueInRow) {
        return searchValue === '' ||
            valueInRow.indexOf(searchValue) !== -1;
    };

    CURRENT_MODULE.hideElement = function (elementSelector, parentSelector) {
        let parent = getParent(parentSelector);
        parent.find(elementSelector).addClass(DISPLAY_NONE_CLASS);
    };

    CURRENT_MODULE.showElement = function (elementSelector, parentSelector) {
        let parent = getParent(parentSelector);
        parent.find(elementSelector).removeClass(DISPLAY_NONE_CLASS);
    };

    function getParent(parentSelector) {
        let parent = $(document);
        if (parentSelector !== undefined) {
            parent = $(parentSelector);
        }

        return parent;
    }

    window.utilitiesModule = CURRENT_MODULE;
})(jQuery);
```
customer-service.js

```javascript
(function ($) {
    const CURRENT_MODULE = {},
        CUSTOMER_URI = 'https://localhost:44313/api/customer';

    CURRENT_MODULE.getCustomers = function (successCallBack) {
        $.get(CUSTOMER_URI)
            .done(function (customerList, textStatus, jqXHR) {
                successCallBack(customerList);
            })
            .fail(function (jqXHR, textStatus, errorThrown) {
                alert('error ' + textStatus + ' ' + errorThrown);
            });
    };
  
    window.customerService = CURRENT_MODULE;
})(jQuery);
```

index.cshtml:

```html
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}
<div class="col">
    <h1>Customer overview</h1>

    @* Searchform *@
    <div id="search-customer">
        <div class="form-group row">
            <label for="searchCustomerName" class="col-sm-2 col-form-label">@Texts.Name</label>
            <div class="col-sm-5">
                <input type="text" class="form-control" id="searchCustomerName" value="">
            </div>
        </div>
        <div class="form-group row">
            <label for="searchEmail" class="col-sm-2 col-form-label">@Texts.Email</label>
            <div class="col-sm-5">
                <input type="text" class="form-control" id="searchEmail" value="">
            </div>
        </div>
        <div class="form-group row">
            <label for="searchCountry" class="col-sm-2 col-form-label">@Texts.Country</label>
            <div class="col-sm-5">
                <input type="text" class="form-control" id="searchCountry" value="">
            </div>
        </div>
        <button id="customerOverviewSearchButton" type="button" class="btn btn-sm btn-primary col-sm-2">@Texts.Search</button>
    </div>

    @* Grid for data *@
    <table class="table mt-2">
        <thead>
            <tr>
                <th scope="col">@Texts.Name</th>
                <th scope="col">@Texts.Email</th>
                <th scope="col">@Texts.Country</th>
            </tr>
        </thead>
        <tbody id="customer-table-body">
            <tr class="loading-row">
                <td colspan="3"><progress></progress></td>
            </tr>
        </tbody>
    </table>

    @* Row templates *@
    <table class="display-none">
        <tr id="data-row-template" class="customer-row">
            <td>{name}</td>
            <td>{email}</td>
            <td>{country}</td>
        </tr>
    </table>
</div>

@section Scripts {
    <environment include="Development">
        <script src="~/js/utilities.js"></script>
        <script src="~/js/customer-service.js"></script>
        <script src="~/js/customer-overview.js"></script>
    </environment>
    <environment exclude="Development">
        <script src="~/js/dist/customer-overview.min.js"></script>
    </environment>
}
```
bundleconfig.json (to generate the minified file in de dist folder)
```json
[
  {
    "outputFileName": "wwwroot/css/site.min.css",
    "inputFiles": [ "wwwroot/css/site.css" ]
  },
  {
    "outputFileName": "wwwroot/js/dist/customer-overview.min.js",
    "inputFiles": [
      "wwwroot/js/utilities.js",
      "wwwroot/js/customer-service.js",
      "wwwroot/js/customer-overview.js"
    ],
    "minify": {
      "enabled": true,
      "renameLocals": true
    },
    "sourceMap": false
  }
]
```
## Separating HTML control events and JavaScript hook-ups

When embedding JavaScript in the HTML pages, the event hookups are usually written directly in the controls, mixing markup with the JavaScript. In my opinion, this makes it harder to find how and where my JavaScript functions are used. 

By hooking up al the events in the JavaScript module there’s no need to embed any JavaScript in the markup for the page. The HTML markup is only used for layout. The functionality and/or interaction logic is completely separate and centralized in the JavaScript modules.

There are still dependencies between the DOM and the JavaScript module but the test framework I have been using (Jest) provides a DOM mock. The DOM mock can be accessed like the browser DOM and you can test the interactions and their results. I'll show examples in my next blog post.

The example page with embedded JavaScript, rewritten with an IIFE is shown below.

Event hookup before (in the html):

```html
<button type="button" class="btn btn-sm btn-primary col-sm-2" onclick="window.customerOverviewModule.filterData()">@Texts.Search</button>
```

Event hookup after refactoring to the IIFE:
```html
<button id="customerOverviewSearchButton" type="button" class="btn btn-sm btn-primary col-sm-2">@Texts.Search</button>
```

```javascript
(function ($, customerService, utilitiesModule) {

     ///...
    
     CURRENT_MODULE.initialize = function() {
        $('#customerOverviewSearchButton').on('click', CURRENT_MODULE.filterData);
        
        CURRENT_MODULE.loadData();
    };

     CURRENT_MODULE.loadData = function () {
         ///...
     };
    
     CURRENT_MODULE.filterData = function () {
     	///...
     };
     
     ///...
     
     window.customerOverviewModule = CURRENT_MODULE;

})(jQuery, window.customerService, window.utilitiesModule);

$(document).ready(function () {
    window.customerOverviewModule.initialize();
});
```
## Separating HTML templating from your JavaScript code

Sometimes you have to create multiple repeating element groups from a JavaScript array. An example is calling an API to retrieve orders then printing a line with each order object. You might define a constant with the HTML template as a string in your JavaScript. For maintainability, code completion and analysis benefits, I find it better to just include a hidden div with the template on the page. You won’t have to deal with escaping reserved characters in a string like quotes and slashes and also gain indenting and linting support from your IDE.

The HTML string used before the refactoring was:

```javascript
  function getDataRows(customerList) {
            const rowTemplate = '<tr class="customer-row"><td>{name}</td><td>{email}</td><td>{country}</td></tr>';
            let dataRows = '<tr class="loading-row"><td colspan="3"><progress></progress></td></tr>';

            $.each(customerList,
                function (index, customer) {
                    dataRows += rowTemplate
                        .replace('{name}', customer.name)
                        .replace('{email}', customer.email)
                        .replace('{country}', customer.country);
                });

            return dataRows;
        }
````
The code using templating from the overview page example as an IIFE  is shown below:

``` html
 @* Grid for data *@
    <table class="table mt-2">
        <thead>
            <tr>
                <th scope="col">@Texts.Name</th>
                <th scope="col">@Texts.Email</th>
                <th scope="col">@Texts.Country</th>
            </tr>
        </thead>
        <tbody id="customer-table-body">
            <tr class="loading-row">
                <td colspan="3"><progress></progress></td>
            </tr>
        </tbody>
    </table>

    @* Row templates *@
    <table class="display-none">
        <tr id="data-row-template" class="customer-row">
            <td>{name}</td>
            <td>{email}</td>
            <td>{country}</td>
        </tr>
    </table>
```

```javascript
 function getDataRows(customerList) {
        const outerHtmlProperty = 'outerHTML';
        const dataRowTemplateId = 'data-row-template';
        const rowTemplate = $('#' + dataRowTemplateId).prop(outerHtmlProperty)
            .replace('id="' + dataRowTemplateId + '"', '');

        let dataRows = $(LOADING_ROW_SELECTOR).prop(outerHtmlProperty);

        $.each(customerList,
            function (index, customer) {
                dataRows += rowTemplate
                    .replace('{name}', customer.name)
                    .replace('{email}', customer.email)
                    .replace('{country}', customer.country);
            });

        return dataRows;
    }
```