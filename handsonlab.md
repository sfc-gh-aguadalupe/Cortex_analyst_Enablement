

## Load Festival Data

For our lab, we will use some made-up festival data. To support this, we will need to load multiple tables. Please go to the folder and get all the files listed there.

[Cortex Analyst Data](https://www.google.com/search?q=/data)

Log into your Snowflake Demo Account and create a space for these tables to live through the UI. Let's create a database we will call it `SI_EVENTS_HOL`:

<img src="images/image005.png" alt="alt text" width="45%">

<img src="images/image006.png" alt="alt text" width="45%">

Let's create the tables we need for this lab. There are a total of 4 tables needed for this lab, and they can all be created the same way by creating a table from a file in Snowflake. Please repeat the following steps for all four tables:

1.  Customers
2.  Contracts
3.  Ticket Sales
4.  Events

We will load these tables into the public schema.

To load a table from a file in Snowsight, go into the `SI_Events` database we created, go to the Public schema, and go into tables, and then click the create button on the top right to create a table from a file:

<img src="images/image007.png" alt="alt text" width="45%">

Name each table the same name as the file you unzipped:

<img src="images/image008.png" alt="alt text" width="45%">

Open the View Options section in the file format area and make sure that the header is the first line of the document:

<img src="images/image010.png" alt="alt text" width="45%">

<img src="images/image011.png" alt="alt text" width="45%">

Repeat this for all 4 files so you have all 4 tables needed for the lab. When you are complete, you should see the following under tables in the public schema:

<img src="images/image013.png" alt="alt text" width="45%">

-----

##  Create a Semantic Model

Let's now build a more intelligent agent that analyzes the data within Snowflake by utilizing our semantic layer.

Lets create a semantic view using cortex analyst:

<img src="images/image021.png" alt="alt text" width="45%">

Go ahead and select create new view:

<img src="images/image023.png" alt="alt text" width="45%">

We can choose the SI\_EVENTS\_HOL database and PUBLIC schema we created earlier. Then we can select Semantic Views and enter the following name and description:

  * **Name:** `music_festival`
  * **Description:** `This has information on music festivals, including locations, events, and ticket sales`

Let's select our tables from the SI\_EVENTS\_HOL data that we imported at the beginning of the lab:

<img src="images/image024.png" alt="alt text" width="45%">

Then we can select the columns to choose. Let's just select all columns by choosing the top checkbox:

<img src="images/image025.png" alt="alt text" width="45%">

### View the Results

When we click 'Create and Save' it will generate a starter semantic model for us:

<img src="images/image026.png" alt="alt text" width="45%">

We can see that it has already created a description as well as some other details, like synonyms. Let's go ahead and add 'show' and 'concert' to the list of synonyms for EVENT\_NAME in the EVENTS table:

<img src="images/image027.png" alt="alt text" width="45%">

### Move Columns

The model incorrectly identified EVENT\_ID as a fact due to its numerical data type, but it is actually a dimensional column. This can be easily corrected by moving it to the dimensions section in the edit menu. It is crucial to review and adjust the model after Snowflake's initial column categorization.

<img src="images/image028.png" alt="alt text" width="45%">

We will need to do this for the following incorrectly identified columns:

1.  TICKET SALES
      * TICKET\_ID - move to dimension
      * EVENT\_ID - move to dimension
      * CUSTOMER\_ID - move to dimension
2.  CUSTOMERS
      * CUSTOMER\_ID - move to dimension
3.  EVENTS
      * EVENT\_ID - move to dimension

We also need to assign unique values option to primary keys. We will do this for the following tables:

1.  Ticket Sales
      * Ticket\_ID
2.  Customers
      * Customer\_ID
3.  Events
      * Event\_ID

This is done by editing each dimension and then checking the box for unique identifier:

<img src="images/image029.png" alt="alt text" width="45%">

Do this for all the listed tables above. Then you will need to save the semantic model and refresh the page.

<img src="images/image030.png" alt="alt text" width="45%">

### Test the Model

I can test this model right now by going to the side window and putting in the following prompt: `What are the different events in Europe`

<img src="images/image031.png" alt="alt text" width="45%">

Then it will go against my model and then write SQL, and then execute that SQL against my model.

### Define Table Relationships

We then need to define our relationships with the other tables. By default, no relationships are created, but they are needed for more complex queries that span multiple tables. When selecting the columns if you do not see your columns you might have missed the step to make ID columns unique on the table please see above. You might also just need to do a full refresh on the page.

Our first relationship we will define is TICKET\_SALES to EVENT. For every EVENT, there are many ticket sales, and the joining column is EVENT\_ID. Let's define a relationship that represents that:

  * **Relationship Name:** `TicketSales_to_Events`
  * **Left Table:** `Ticket_Sales`
  * **Right Table:** `Events`
  * **Relationship Columns:** `EVENT_ID` and `EVENT_ID`

<img src="images/image032.png" alt="alt text" width="45%">

Let's add a second relationship of customers to tickets:

  * **Relationship Name:** `TicketSales_to_Customers`
  * **Left Table:** `Ticket_Sales`
  * **Right Table:** `Customers`
  * **Relationship Columns:** `CUSTOMER_ID` and `CUSTOMER_ID`

<img src="images/image033.png" alt="alt text" width="45%">

<img src="images/image034.png" alt="alt text" width="45%">

Now we have two relationships set up in our semantic model. Let's go ahead and test them using the prompt on the right side. We can ask this: `How many ticket sales were there for Difficult Fest?`

<img src="images/image035.png" alt="alt text" width="45%">

We can see that it was able to utilize our join and see how many tickets were sold for that event by joining the two tables together we defined in our relationship.

We can also ask it to `Count the total customers by customer region that went to Difficult Fest`, and this should span all of our tables in our semantic model and give us values:

<img src="images/image036.png" alt="alt text" width="45%">

Excellent\! This functionality is operating as anticipated.

### Verify the Queries

Since these queries returned the correct values, we can add them as verified queries, which will fuel our model's improvement.

<img src="images/image037.png" alt="alt text" width="45%">

<img src="images/image038.png" alt="alt text" width="45%">

These will now show up under verified queries in our model definition:

<img src="images/image039.png" alt="alt text" width="45%">

### Save the Model

When you are done, make sure you save the model:

<img src="images/image040.png" alt="alt text" width="45%">
