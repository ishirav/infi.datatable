Overview
========
This project provides Backbone/Bootstrap components for displaying Infinidat-style collections.
It uses the server's REST API for sorting and pagination.

Features
--------
* Plain ol' HTML table that uses Bootstrap styles.
* Table data is loaded from the server via AJAX.
* Support for custom rendering of table cells.
* Single-column sorting.
* Pagination.
* Simple or advanced search/filter.
* The user can choose which columns to display, and the selection is saved in session storage.
* Full BACK button support.

Not supported yet: multi-column sorting; column resize; column reorder.

Dependencies
------------
* JQuery 1.11+ (https://jquery.com/)
* Underscore.js 1.8+ (http://underscorejs.org/)
* Backbone.js 1.2+ (http://backbonejs.org/)
* Bootstrap 3.x (http://getbootstrap.com/)
* Bootpag 1.0.7+ (http://botmonster.com/jquery-bootpag/) - required for DataTablePaginator
* jQuery QueryBuilder 2.3+ (http://mistic100.github.io/jQuery-QueryBuilder/) - required for DataTableQueryBuilder


Classes
=======

### DataTableCollection

Use this collection for working with Infinidat-style REST APIs. Such APIs return responses in the following format:

```javascript
{
    "result": [
        {...},
        {...},
        {...}
    ],
    "metadata": {
        "ready": true,
        "number_of_objects": 3,
        "page_size": 50
        "pages_total": 1,
        "page": 1
    },
    "error": null
}
```

At a minimum, provide the URL to load the collection from:

```javascript
var Events = DataTableCollection.extend({
    url: 'http://localhost/api/rest/events/'
});

var events = new Events();
```

You may also want to override the `page_size` property to increase it from the default value of 10,
and the `get_request_headers` function if any custom headers should be sent to server when loading the collection.

**Note:** currently there can be only one `DataTableCollection` on the page, since it uses the page URL to save its state (for BACK button support).

### DataTable

This is the main view class that takes a `DataTableCollection` and displays it in an HTML table.
Here is a skeleton of the code needed for creating it:

```javascript
var dt = new DataTable({
    id: '...',
    collection: events,
    columns: [
        {...},
        {...},
        {...}
    ],
    row_click_callback: function(model) {
    }
});
$('.container').append(dt.el);
```

The required properties are `id`, `collection` and `columns`. The `columns` list defines the table columns,
and each column can have the following properties:

* **name** - this is the only required property. It typically matches one of the attribute names in the collection's models.
* **title** - the column title. If not specified, it is created based on the name.
* **width** - the column width, either as number of pixels or as a CSS length expression (e.g. "10em").
  If not specified the width will be set to "auto".
* **visible** - whether the column will be visible by default. The table may have several hidden columns,
  that the user can manually show. Defaults to true.
* **sortable** - whether the column can be sorted. Defaults to true.
* **render** - defines a custom function for rendering the cell's contents to HTML. The function receives an object with
  three properties: `model`, `column` and `value`, and should return the formatted value.

The optional `row_click_callback` can be used for handling clicks on the table rows.
It gets the model displayed by the clicked row.

#### Styling the table

The DOM which is generated is a plain HTML table, so it is easy to style using CSS.
By default the table gets assigned the Bootstrap classes `table`, `table-hover` and `table-bordered`
but you can change this by overriding the view's `className` property.

Each table cell gets a class derived from its column name. For example if the column name is "timestamp",
the cells' class will be `th_timestamp` and `td_timestamp`. This makes it easy to style each column separately.

Table header cells also get the classes `sortable` (if the column is sortable), `asc` (when sorted in ascending order)
and `desc` (when sorted in descending order).


### DataTablePaginator

A view that uses Bootpag (http://botmonster.com/jquery-bootpag/) for paginating through the collection.
Use it like this:

```javascript
var paginator = new DataTablePaginator({
    collection: events
});
$('.container').append(paginator.el);
```


### DataTableCounter

A view that displays the total number of items in the collection (taken from the `number_of_objects` metadata field).
It also supports the `limited_number_of_objects` and `approximated_number_of_objects` boolean flags, should those
exist in the metadata.

Usage example:
```javascript
var counter = new DataTableCounter({
    collection: events
});
$('.container').append(counter.el).append(' total events');
```


### DataTableSimpleQuery

Use this view to display a single search field for filtering the collection. The contents of the field is sent
to the collection by calling its `set_filters` function, and this triggers a reloading of the collection from the server. It is up to the server to filter the collection according to the search terms.

Usage example:
```javascript
var search = new DataTableSimpleQuery({
    collection: events,
    field_name: 'search_terms'
});
search.render();
$('#search_container').html(search.el);
```

The optional `field_name` specifies the name of the query parameter which will be sent to the server. The default name is `q`.

### DataTableQueryBuilder

A more advanced filtering view that uses jQuery QueryBuilder (http://mistic100.github.io/jQuery-QueryBuilder/).
It allows the user to build a filtering expression on one or more fields in the collection.

For example:
```javascript
var qb = new DataTableQueryBuilder({
    collection: events,
    filter_fields: [
        {id: 'timestamp', type: 'datetime'},
        {id: 'seqnum', type: 'integer'},
        {id: 'description', type: 'string'},
        {id: 'level', type: 'string', value: ['INFO', 'WARNING', 'ERROR'], input: 'radio'}
    ]
});
$('#search_container').html(qb.el);
qb.render();
```

The `filter_fields` array defines which fields can be filtered on. Refer to the jQuery QueryBuilder documentation
for details about the options available when defining such fields.
