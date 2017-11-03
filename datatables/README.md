# Use DataTables with Chisel

[DataTables](https://datatables.net) is a popular jQuery plugin to enhance your tables functionality with dynamic sorting, filtering and much more.

## Prerequisites

Make sure you have created a project with jQuery support.

## Step by step installation

### 1. Install DataTables via npm

```
npm install --save-dev datatables.net datatables.net-src
```

**Note**: Yes, unfortunately you need two different versions. First one is for JavaScript related part. The second one is for SCSS usage.

### 2. JavaScript code sample

Inside ``src/scripts`` create a file e.g. ``datatabales.js`` and use this basic code.

```javascript
const $ = require('jquery');
const DataTables = require('datatables.net');
$.fn.DataTables = DataTables();

$('.js-datatable').each(function () {
  $(this).DataTables();
});
```

**Notes**: 
1. I'm using ``.js-datatables`` jQuery selector here. You might want to use something else.
2. It's just a basic implementation. No extra settings passed on the initialization.

### 3. Include your file in the app.js

```javascript
require('./datatables');
```

### 4. (S)CSS part

Inside your ``src/styles`` create a vendor directory and create a file e.g. ``_datatables.scss`` and import the source file.

```sass
@import "datatables.net-src/css/jquery.DataTables";
```

Make sure the ``vendor`` directory is imported in the ``src/styles/main.scss``. We recommend to place it below the components import.

```sass
...
@import 'components/*';
@import 'vendor/*'; // <-- OUR VENDOR
@import 'utilities/*';
```

Here's an example of the ``vendor/_datatables.scss`` partial file:

```scss
// alter default settings
$table-header-border: none;
$table-paging-button-active: transparent;
$table-paging-button-hover: transparent;
$table-shade: transparent;

@import "datatables.net-src/css/jquery.dataTables";

// put your overriding below
```

**Note**: We recommend checking the content of ``datatables.net-src/css/jquery.DataTables.scss`` inside your ``node_modules``. You may find some interesting settings there which can help you style your table. Knowing the variables you can change the default look of the tables created using dataTables plugin. 
