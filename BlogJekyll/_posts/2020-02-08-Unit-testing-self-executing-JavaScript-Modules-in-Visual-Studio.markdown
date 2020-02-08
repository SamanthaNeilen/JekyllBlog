---
layout: post
title:  "Unit testing self-executing JavaScript modules in Visual Studio"
date:   2020-02-08 00:00:00 +0100

tags: JavaScript TestAutomation
---
Visual Studio has support for unit tests in the Test Explorer window so you can run all your C# and JavaScript tests from that editor. A separate Node project can contain al the test code separate from the code that gets published and deployed. It is possible to include the tests in the web project itself. See [the Microsoft Docs page on Unit testing JavaScript and TypeScript in Visual Studio](https://docs.microsoft.com/en-us/visualstudio/javascript/unit-testing-javascript-with-visual-studio?view=vs-2019#unit-tests-in-other-project-types) for more details.

This post is a continuation of my post about [Creating more maintainable JavaScript with self-executing functions](https://samanthaneilen.github.io/2019/12/30/Creating-more-maintainable-javascript-with-self-executing-functions.html). I will continue building on the JavaScriptModules project presented in that post. A working Visual Studio project with the code can be found in my [JavaScriptModuleExamples repository on GitHub](https://github.com/SamanthaNeilen/JavascriptModulesExamples).
 If you use the project from GitHub be sure to follow to ReadMe.md on the repository for additional steps to see the tests in Visual Studio Test Explorer. 

The befits of writing these JavaScript unit tests are getting earlier feedback about the state of your JavaScript without writing more fragile and slower integration tests. The unit tests can also be run in the build pipeline next to your regular unit tests using an npm test command step before deploying to an environment. 

**Table of contents:**
* Table of Contents
{:toc}

### Setup a Node.Js Test project

So to start testing, add a Blank Node.js Console Application project. You need to have the Node.js workload installed in Visual Studio. When you add a new file to the project via the “Add -> New Item” context menu, you can see the test frameworks that Visual Studio already provides a template file for. The template file contains a passing test and a failing test showing the syntax for basic tests. 

![[JavaScript unit test file templates]]({{"/assets/images/20200208/JavaScriptUnitTestFileTemplates.png" | relative_url }})

[An article comparing several of the different JavaScript frameworks can be found on Medium.](https://medium.com/welldone-software/an-overview-of-javascript-testing-in-2019-264e19514d0a) 

I will be working with the Jest framework in this post, so add a Jest test file. The contents for the default new Jest file is shown below. 

``` javascript
describe('Test Suite 1', function () {
    it("Test 1 - This shouldn't fail", function () {
        expect(true).toBeTruthy();
    });

    it('Test 2 - This should fail', function () {
        expect(1 === 1).toBeTruthy(); // "This shouldn't fail"
        expect(false).toBeTruthy();
    });
});
```

To run the tests using Visual Studio Test you need to set the “JavaScript Unit Test” properties in the project properties and install the jest and jest-editor-support npm packages.

For the “JavaScript Unit Tests” properties set the “Test Framework” to the framework of choice (Jest, in my case) and set the Test Root to a “Tests” subfolder. Using a subfolder will ensure that you can add extra folders with builder and helper classes without them being seen as potential unit test files.

Now the tests will show up in Test Explorer after building the project. You may have to restart Visual Studio the first time before any tests show up. Also to ensure running or debugging the latest version of your tests you always need to build before running a test via Test Explorer.

![[First tests loaded in Test Explorer]]({{"/assets/images/20200208/FirstTestsLoadedInTestExplorer.png" | relative_url }})

### Writing tests

To start testing the customer-overview.js file from the JavaScriptModules web project we first have to hookup some dependencies and mocks. We need jQuery, a mock of our customer-service.js and the HTML or DOM setup that the module can interact with.

First, add a testutilities.js file that will set up a module that has the link to the wwwroot of our project under test and loads in 3rd party libraries needed for testing. The module is shown below. 

``` javascript
(function() {
    let currentModule = {};
    let wwwRootScriptsFolder = '../JavaScriptModules/wwwroot/';

    currentModule.wwwRoot = wwwRootScriptsFolder;

    currentModule.initialize = function () {
        global.jQuery = global.$ = require(wwwRootScriptsFolder + 'lib/jquery/jquery.js');
    };

    global.TestUtilitiesModule = currentModule;
})();
```

Now we can use the TestUtilities module in the test file to do a simple test using jQuery. This example also shows how to declare some HTML to interact with. The document is actually a mock provided by the test framework. Any interactions your code and modules have with the window or document will work as expected.

``` javascript
describe('Test Suite 1', function () {
    beforeEach(() => {
        jest.resetModules();
        require('../TestUtilities.js');
        global.TestUtilitiesModule.initialize();

        document.body.innerHTML = '';
        const input = document.createElement('label');
        input.id = 'myLabel';
        document.body.appendChild(input);
    });

    it("Test 1 - This shouldn't fail", function () {
        expect($('#myLabel')).toHaveLength(1);
    });

    it('Test 2 - This should fail', function () {
        expect(1 === 1).toBeTruthy(); // "This shouldn't fail"
        expect(false).toBeTruthy();
    });
});

```
If you run the test in debugging mode, you can stop and inspect the variables and DOM.

![[Debugging JavaScript test]]({{"/assets/images/20200208/DebugTest.png" | relative_url }})

Now, we want to build the HTML template for the initial page for the module under test. I usually create a module using [the builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) to be able to create the template in a fluent syntax. This way you can use the builder in your tests to create multiple starting HTML situations with expressive and readable syntax. The builder and the test file using the builder are shown below. Not that in the builder below I only add the required elements to check my logic. All elements that are for layout and not functionality are omitted. (The code below can be also be found in my GitHub repo under the tag: V1.3-first-test-module-using-HTML-builder [my GitHub repo under the tag: V1.3-first-test-module-using-HTML-builder](https://github.com/SamanthaNeilen/JavascriptModulesExamples/tree/V1.3-first-test-module-using-HTML-builder).)

(Also note that the code shown is a refinement of multiple times writing, evaluating, refactoring and running and reading the tests iterations. If your logic and HTML are simple you may not even these builders to write a few simple tests.)

UnitTest.js:
``` javascript
describe('Search using Name', function () {
    beforeEach(() => {
        const testProjectRoot = '../../../';
        jest.resetModules();

        require(testProjectRoot + 'TestUtilities.js');
        global.TestUtilitiesModule.initialize();
        require(testProjectRoot + 'Builders/CustomerOverviewHtmlBuilder.js');

        let wwwRoot = testProjectRoot + global.TestUtilitiesModule.wwwRoot;
        require(wwwRoot + 'js/utilities.js');
        require(wwwRoot + 'js/customer-overview.js');

        //arrange initial state
        let htmlTemplate = global.customerOverviewHtmlBuilderModule
            .new()
            .clearDataRows()
            .hideLoadingRow()
            .withDataRow('first customer', 'first customer email', 'first customer country')
            .withDataRow('second customer', 'second customer email', 'second customer country')
            .withDataRow('testcustomer', 'testcustomer email', 'testcustomer country')
            .build();

        document.body.innerHTML = '';
        document.body.appendChild(htmlTemplate);
    });

    it("WHEN 3 rows loaded THEN Search with full Name SHOULD filter to show one row", function () {
        //arrange
        let searchNameValue = 'second customer';
        let initialVisibleRows = getVisibleRows();
        let initialInvisibleRows = getInvisibleRows();

        //act
        $("#searchCustomerName").val(searchNameValue);
        window.customerOverviewModule.filterData();

        //assert intial state
        expect(initialVisibleRows).toHaveLength(3);
        expect(initialInvisibleRows).toHaveLength(1);

        //assert state after action
        let filteredVisibleRows = getVisibleRows();
        let filteredInvisibleRows = getInvisibleRows();
        
        expect(filteredVisibleRows).toHaveLength(1);
        expect(filteredInvisibleRows).toHaveLength(3);

        let actualVisibleRowSearchName = getSearchNameValueForSingleRow(filteredVisibleRows[0]);
        expect(actualVisibleRowSearchName).toBe(searchNameValue);
    });

    function getSearchNameValueForSingleRow(row) {
        return $(row).find('td')[0].innerHTML;
    }

    function getVisibleRows() {
        
        return $.grep(getAllRows(), function (row) { return !$(row).hasClass('display-none'); });
    }

    function getInvisibleRows() {
        return $.grep(getAllRows(), function (row) { return $(row).hasClass('display-none'); });
    }

    function getAllRows() {
        return $('#customer-table-body').find('tr');
    }
});

```

TestUtilities.js:
``` javascript
(function() {
    let currentModule = {};
    let wwwRootScriptsFolder = '../JavaScriptModules/wwwroot/';

    currentModule.wwwRoot = wwwRootScriptsFolder;

    currentModule.initialize = function () {
        global.jQuery = global.$ = require(wwwRootScriptsFolder + 'lib/jquery/jquery.js');
        require('./Builders/BaseHtmlBuilder.js');
    };
    
    global.TestUtilitiesModule = currentModule;
})();
```

CustomerOverviewHtmlBuilder.js:
``` javascript
(function ($, baseHtmlBuilder) {
    let currentModule = {};
    let root = document.createElement('div');

    currentModule.new = function () {
        root = document.createElement('div');

        let searchForm = createSearchForm();
        root.appendChild(searchForm);

        let dataGrid = createEmptyGrid();
        root.appendChild(dataGrid);

        let rowTemplates = createRowTemplates();
        root.appendChild(rowTemplates);

        return currentModule;
    };


    currentModule.clearDataRows = function() {
        let customerTable = $(root).find('#customer-table-body');
        let loadingRow = customerTable.find('.loading-row');
        customerTable.html('');
        customerTable.append(loadingRow);

        return currentModule;
    };

    currentModule.hideLoadingRow = function () {
        let customerTable = $(root).find('#customer-table-body');
        let loadingRow = customerTable.find('.loading-row');
        loadingRow.addClass('display-none');

        return currentModule;
    };
    
    currentModule.withDataRow = function (name, email, country) {
        let customerTable = $(root).find('#customer-table-body');
        let row = baseHtmlBuilder.createRow(
            {
                Id: '',
                className: 'customer-row',
                cells: [
                    { innerHTML: name },
                    { innerHTML: email },
                    { innerHTML: country }]
            }
        );
        customerTable.append(row);

        return currentModule;
    };

    currentModule.build = function () {
        return root;
    };

    function createSearchForm() {
        let searchform = document.createElement('div');
        searchform.id = 'search-customer';

        searchform.appendChild(baseHtmlBuilder.createTextInput('searchCustomerName'));
        searchform.appendChild(baseHtmlBuilder.createTextInput('searchEmail'));
        searchform.appendChild(baseHtmlBuilder.createTextInput('searchCountry'));
        searchform.appendChild(baseHtmlBuilder.createButton('customerOverviewSearchButton'));

        return searchform;
    }

    function createEmptyGrid() {
        let table = document.createElement('table');

        let tableBody = document.createElement('tbody');
        tableBody.id = 'customer-table-body';
        table.appendChild(tableBody);

        let row = document.createElement('tr');
        row.className  = 'loading-row';
        tableBody.appendChild(row);

        return table;
    }

    function createRowTemplates() {
        let table = document.createElement('table');
        table.className = 'display-none';

        let row = baseHtmlBuilder.createRow(
            {
                id: 'data-row-template',
                className: 'customer-row',
                cells: [
                    { innerHTML: '' },
                    { innerHTML: '' },
                    { innerHTML: '' }]
            }
        );
        table.appendChild(row);
        
        return table;
    }

    global.customerOverviewHtmlBuilderModule = currentModule;
})(jQuery, global.BaseHtmlBuilderModule);
```
BaseHtmlBuilder.js:
``` javascript
(function ($) {
    let currentModule = {};
    currentModule.createTextInput = function (id) {
        let input = document.createElement('input');
        input.id = id;
        input.type = 'text';
        return input;
    };

    currentModule.createButton = function (id) {
        let button = document.createElement('button');
        button.id = id;
        button.type = 'button';
        return button;
    };

    currentModule.createRow = function (rowDefinition) {
        let rowElement = document.createElement('tr');
        rowElement.id = rowDefinition.Id;
        rowElement.className = rowDefinition.className;

        let cells = currentModule.createCells(rowDefinition.cells);
        $.each(cells,
            function (index, cell) {
                rowElement.appendChild(cell);
            });

        return rowElement;
    };

    currentModule.createCells = function (cells) {
        let resultingCells = [];

        $.each(cells,
            function (index, cell) {
                let cellElement = document.createElement('td');
                cellElement.className = cell.className;
                cellElement.innerHTML = cell.innerHTML;
                resultingCells.push(cellElement);
            });

        return resultingCells;
    };

    global.BaseHtmlBuilderModule = currentModule;
})(jQuery);
```
When you see this code it might seem like a lot of work to create a few tests. By extracting helper methods to a separate class you will have to invest a bit more initially but will be faster at writing tests later. Also by practicing TDD, you should be able to create the template as you are building your HTML and initial JavaScript methods so the extra invested time should not come all at once after completing your complete HTML and JavaScript logic.

A debugging/troubleshooting tip for your HTML builders. If you’re unsure that the code to create the HTML is correct. At a watch to the document.body.innerHTML variable and paste the value in a new HTML page file in Visual Studio. Select all contents in the file and right-click to get the context menu and select Un-Minify to get a nice readable HTML with syntax highlighting to troubleshoot the output of the builder.

![[Un-Minify option for HTML]]({{"/assets/images/20200208/DocumentHtmlUnMinify.png" | relative_url }})

Jest supports a way to use tests with parameters but the Visual Studio integration does not pick them up. So for now, I just add multiple tests for the different parameters I would like to test. To avoid having to change all the tests when my arrangements or assertions have to change, I introduce ScenarioBuilder and/or AssertionHelper modules that again abstract the test scenarios and assertions in a nice readable fluent syntax. 

In the example, I added a scenario builder module in the file itself containing a full set of arrange, act and assert methods. If the module gets to large or you see duplication across several builders you could choose to create builders with better separation of concerns. The point is to show an example of a pattern for writing readable tests where the implementation details can be changed in a single spot in the code should the page or functionality change. For the full working solution see [my GitHub repo with the tag: V1.4-first-test-using-scenario-builder](https://github.com/SamanthaNeilen/JavascriptModulesExamples/tree/V1.4-first-test-using-scenario-builder).)

UnitTest.js
``` javascript
describe('Search using Name', function () {

    it("WHEN 3 rows loaded THEN Search with full Name SHOULD filter to show one row", function () {
        let scenario = global.SearchCustomerNameScenarioBuilderModule.new();

        //arrange
        const valueToSearch = 'second customer';
        const expectedNumberOfVisibleRows = 1;
        const expectedNumberOfInvisibleRows = 3;
        
        scenario.arrange_Page_With_3_Initial_DataRows();

        //act
        scenario.act_SearchName(valueToSearch);

        //assert
        scenario.assert_Number_Of_Visible_Rows(expectedNumberOfVisibleRows)
            .assert_Number_Of_Invisible_Rows(expectedNumberOfInvisibleRows)
            .assert_Visible_Rows_Name_Is_Complete_SearchValue(valueToSearch)
            .assert_Invisible_Rows_Name_Do_Not_Contain_SearchValue(valueToSearch);
    });

    it("WHEN 3 rows loaded THEN Search with partial Name SHOULD filter to show one row", function () {
        let scenario = global.SearchCustomerNameScenarioBuilderModule.new();

        //arrange
        const valueToSearch = 'test';
        const expectedNumberOfVisibleRows = 1;
        const expectedNumberOfInvisibleRows = 3;

        scenario.arrange_Page_With_3_Initial_DataRows();

        //act
        scenario.act_SearchName(valueToSearch);

        //assert
        scenario.assert_Number_Of_Visible_Rows(expectedNumberOfVisibleRows)
            .assert_Number_Of_Invisible_Rows(expectedNumberOfInvisibleRows)
            .assert_Visible_Rows_Name_Contains_SearchValue(valueToSearch)
            .assert_Invisible_Rows_Name_Do_Not_Contain_SearchValue(valueToSearch);
    });

    it("WHEN 3 rows loaded THEN Search with empty Name SHOULD filter to show all rows", function () {
        let scenario = global.SearchCustomerNameScenarioBuilderModule.new();

        //arrange
        const valueToSearch = '';
        const expectedNumberOfVisibleRows = 3;
        const expectedNumberOfInvisibleRows = 1;

        scenario.arrange_Page_With_3_Initial_DataRows();

        //act
        scenario.act_SearchName(valueToSearch);

        //assert
        scenario.assert_Number_Of_Visible_Rows(expectedNumberOfVisibleRows)
            .assert_Number_Of_Invisible_Rows(expectedNumberOfInvisibleRows);
    });


    (function (jest, expect) {
        let currentModule = {};

        currentModule.new = function() {
            const testProjectRoot = '../../../';
            jest.resetModules();

            require(testProjectRoot + 'TestUtilities.js');
            global.TestUtilitiesModule.initialize();
            require(testProjectRoot + 'Builders/CustomerOverviewHtmlBuilder.js');

            let wwwRoot = testProjectRoot + global.TestUtilitiesModule.wwwRoot;
            require(wwwRoot + 'js/utilities.js');
            require(wwwRoot + 'js/customer-overview.js');

            return currentModule;
        };

        currentModule.arrange_Page_With_3_Initial_DataRows = function () {
            document.body.innerHTML = '';

            const template = global.customerOverviewHtmlBuilderModule
                .new()
                .clearDataRows()
                .hideLoadingRow()
                .withDataRow('first customer', 'first customer email', 'first customer country')
                .withDataRow('second customer', 'second customer email', 'second customer country')
                .withDataRow('testcustomer', 'testcustomer email', 'testcustomer country')
                .build();

            document.body.appendChild(template);

            return currentModule;
        };

        currentModule.act_SearchName = function (nameToSearch) {
            $("#searchCustomerName").val(nameToSearch);
            window.customerOverviewModule.filterData();

            return currentModule;
        };

        currentModule.assert_Number_Of_Visible_Rows = function (numberOfRows) {
            expect(getVisibleRows()).toHaveLength(numberOfRows);

            return currentModule;
        };

        currentModule.assert_Number_Of_Invisible_Rows = function (numberOfRows) {
            expect(getInvisibleRows()).toHaveLength(numberOfRows);

            return currentModule;
        };

        currentModule.assert_Visible_Rows_Name_Is_Complete_SearchValue = function (nameToSearch) {
            let rows = getVisibleRows();
            $.each(rows,
                function (index, row) {
                    if (!isLoadingRow(row)) {
                        expect($(row).find('td')[0].innerHTML).toBe(nameToSearch);
                    }
                });

            return currentModule;
        };

        currentModule.assert_Visible_Rows_Name_Contains_SearchValue = function (nameToSearch) {
            let rows = getVisibleRows();
            $.each(rows,
                function (index, row) {
                    if (!isLoadingRow(row)) {
                        expect($(row).find('td')[0].innerHTML).toEqual(expect.stringContaining(nameToSearch));
                    }
                });

            return currentModule;
        };

        currentModule.assert_Invisible_Rows_Name_Do_Not_Contain_SearchValue = function (nameToSearch) {
            let rows = getInvisibleRows();
            $.each(rows,
                function (index, row) {
                    if (!isLoadingRow(row)) {
                        expect($(row).find('td')[0].innerHTML).toEqual(expect.not.stringContaining(nameToSearch));
                    }
                });

            return currentModule;
        };

        function isLoadingRow(row) {
            return $(row).hasClass('loading-row');
        }

        function getVisibleRows() {
            return $.grep(getAllRows(), function (row) { return !$(row).hasClass('display-none'); });
        }

        function getInvisibleRows() {
            return $.grep(getAllRows(), function (row) { return $(row).hasClass('display-none'); });
        }

        function getAllRows() {
            return $('#customer-table-body').find('tr');
        }

        global.SearchCustomerNameScenarioBuilderModule = currentModule;

    })(jest, expect);
});

```

### Resources
* [Microsoft Docs page on Unit testing JavaScript and TypeScript in Visual Studio](https://docs.microsoft.com/en-us/visualstudio/javascript/unit-testing-javascript-with-visual-studio?view=vs-2019#unit-tests-in-other-project-types)
* [Jest expect method documentation](https://jestjs.io/docs/en/expect)
* [Wikipedia page for the builder pattern](https://en.wikipedia.org/wiki/Builder%20pattern)