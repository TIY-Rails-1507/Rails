# Updating the view challenge
Change the Index view so that it uses an HTML table to display the data.

The recommended way is to build a static HTML table first. Then embed Ruby where needed, to make it dynamic. 

They key elements of an HTML table are:
* table - defines an html table
* tr - defines a table row
* td - defines a cell, an intersection of a row and column

A write up on HTML tables: http://www.simplehtmlguide.com/tables.php

Basic HTML table example:
```html
<table border="1">
  <tr>
    <td>Jill</td>
    <td>Smith</td> 
    <td>50</td>
  </tr>
  <tr>
    <td>Eve</td>
    <td>Jackson</td> 
    <td>94</td>
  </tr>
</table>
```
Adapted from: http://www.w3schools.com/html/html_tables.asp

HTML example with header, footer and body
```html
<table border="1">
  <thead>
  <tr>
     <th>Month</th>
     <th>Savings</th>
  </tr>
  </thead>
  <tfoot>
  <tr>
      <td>Sum</td>
      <td>$180</td>
  </tr>
  </tfoot>
  <tbody>
  <tr>
     <td>January</td>
     <td>$100</td>
  </tr>
  <tr>
      <td>February</td>
      <td>$80</td>
  </tr>
  </tbody>
</table>
```

Ref: http://www.w3schools.com/tags/tag_tbody.

Hint: If you use the second example, you may want to drop the footer tag for this exercise...

Note we are giving these tables a border of 1. This is not good practice, we should be using CSS. We will change this at a later stage. 
