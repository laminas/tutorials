# Getting Started with Laminas MVC Applications

This tutorial is intended to give an introduction to using Laminas by creating a simple database driven application using the Model-View-Controller paradigm. By the end you will have a working Laminas application and you can then poke around the code to find out more about how it all works and fits together.

## The tutorial application

The application that we are going to build is a simple inventory system to
display which albums we own. The main page will list our collection and allow us
to add, edit and delete CDs. We are going to need four pages in our website:

Page           | Description
-------------- | -----------
List of albums | This will display the list of albums and provide links to edit and delete them. Also, a link to enable adding new albums will be provided.
Add new album  | This page will provide a form for adding a new album.
Edit album     | This page will provide a form for editing an album.
Delete album   | This page will confirm that we want to delete an album and then delete it.

We will also need to store our data into a database. We will only need one table
with these fields in it:

Field name | Type         | Null? | Notes
---------- | ------------ | ----- | -----
id         | integer      | No    | Primary key, auto-increment
artist     | varchar(100) | No    |
title      | varchar(100) | No    |
