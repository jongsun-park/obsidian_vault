
https://www.udemy.com/course/sql-query-training-for-sap-business-one/learn/lecture/9239714#overview

Based on my video you need a few things to get going.

## **Notepad2 - My Favourite Text Editor**

I use Notepad2 for my text editing.  Do NOT use Microsoft Word for coding.  It adds all sorts of formatting and is unacceptable as a tool for building queries.  Also, do NOT use the stock Windows text editor.  It has only a single "undo" instance, no syntax highlighting and no additional features to help indenting, etc.

1. Download Notepad2 from [http://www.flos-freeware.ch/notepad2.html](http://www.flos-freeware.ch/notepad2.html)
2. I suggest the setup version: [click here (32-bit)](http://www.flos-freeware.ch/zip/Notepad2_4.2.25_x86.exe) [click here (64-bit)](http://www.flos-freeware.ch/zip/Notepad2_4.2.25_x64.exe)
3. Install it and it will automatically prompt you to replace Windows text editor.  I strongly recommend you push "Yes"
4. (Optional) If you manually install a different version then you would need to put the .exe file somewhere first
5. (Optional) Then if manually installed you need to right-click a .txt file >> select "Open with" then select "Choose another app".  Then you need to locate where you put your Notepad2 files and select the .exe.  It's much easier to use my setup link from step two.

Now you can right-click your desktop or in a folder and select New >> Text Document and you are on your way.  As you start typing click F12 and select SQL Query to set the syntax highlighting.

This will be a giant help as you move along.

## **Microsoft SQL Management Studio - Your Bridge to SQL Town**

I use MSSMS for my editing and creation of new queries.  It's also fantastic for debugging.  I will generally store the final version in a text file so I don't lose it, but my work, debugging and experimentation will be done in SQL.

You don't need to "download" it, but you need to request access to it in order to use it.  This may require installing it on your local machine if you are in the network or connecting via RDP to the server.  This course will not cover getting server access to the location, we will assume that you have asked an IT person for the location and login information of SQL.

1. Run MS SQL Management Studio
2. Login using your credentials
3. Click the plus sign next to the "Databases" item on the "Object Explorer"
4. Right-click your production database and select "New Query".  If you do not know what your production database is.  Go back to SAP Business One and click Administration >> Choose Company.  You will see a list of company names and their database names.
5. Now you can paste your new query into the query window.  NOTE: Avoid anything that has DELETE, UPDATE or ALTER in the query as this can cause serious damage to the database.
6. I would recommend turning line numbers on by clicking Tools >> Options >> Text Editor >> All Languages >> General and then check the box for "Line numbers".  This will help when debugging.
7. When ready to run your query click F5 or click the "Execute" button and this will run your query and show you the results (or errors).

As you are running and debugging be careful with your syntax and read the debugging error messages carefully.  Google what you don't know and continue with this course, I'll explain what you need to understand to build well structured queries.


## SAP Query Generator

Useful for gathering column names and for saving back into SAP.  Other than that, it's a useless tool that's dangerous because it will cause you to frequently lose data.  Load in your tables and you will see columns and related tables (bold column names have related tables, left-click and hold and then drag the field back onto the table area and it will join the tables, see my "Finding Data" lecture for the video).

1. Login to SAP
2. Click Tools >> Queries >> Query Generator
3. Click in the different fields for "Select", "Where", etc. and then you can double click the columns as needed to add them.
4. When you have the columns that you need just click "Execute"
5. The resulting query and data (or errors) will be shown
6. Copy the query out to Notepad2.  I do not recommend working in Query Preview or Generator.

![](https://img-c.udemycdn.com/redactor/raw/2018-01-31_07-06-55-1e1c48b0e2ca31126ee445b52c3b2064.png)

## Summary

You should now have the basic tools you need in order to begin building all of your queries.  Next step is understanding the structure and syntax of an SQL statement.