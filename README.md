Electronics Sales Analysis
About the project
This project reads the electronics sales CSV, cleans the quantity and price columns, and calculates sales revenue. It uses the built-in csv module for the main analysis. I use pandas at the end to check whether the results agree.
Reading the file
I used csv.DictReader because it stores each sale as a dictionary. This makes it easier to refer to a value by its column name, for example row["quantity"].

The file has 50 records and these columns: sale_id, sale_date, store, product, quantity, unit_price_kes, and payment_method.
Cleaning the data
The parse_number() function changes numeric text into a float. It also removes a KES label and correctly placed thousands commas when they appear in the price column.

I made a few decisions about the unusual values in this dataset:
Sale ID	Original value	What I did	Reason
S-3006	two in quantity	Changed it to 2	It is a clear number word and the column is for item counts.
S-3017	38,999 in unit price	Changed it to 38999	The comma is being used as a thousands separator.
S-3029	KES 18999 in unit price	Changed it to 18999	I removed the currency label but kept the amount.


I skipped three rows because a required numeric value was not available:
Sale ID	Problem	Reason for skipping
S-3030	Unit price is N/A	There is no price to use in the revenue calculation.
S-3035	Quantity is blank	I cannot tell how many items were sold.
S-3039	Unit price is blank	The unit price is unknown.


I did not replace these missing values with zero or an average. Doing that would mean guessing the value of a sale and could change the revenue totals. If either quantity or price is invalid, I skip the whole transaction and keep its sale ID and reason in skipped_rows.

Quantity must be greater than zero and must be a whole number because the data records physical items. A price cannot be negative, but I allow a price of zero because an item could have been given away as a promotion. A missing product, store or payment method is labelled Unknown instead of dropping a sale that has valid quantity and price values.

After cleaning, 47 transactions are used and 3 are skipped.
Revenue calculations
For each sale, revenue is:

quantity * unit_price

The total revenue is the sum of the revenue from all valid sales. I format the amount in KES with commas and two decimal places.

For revenue_by(), I used a normal Python dictionary. The function adds each sale's revenue to the correct product, store or payment method. I did not use Counter or defaultdict.
Top three products
top_n() sorts the dictionary items by revenue from highest to lowest. When two products have the same revenue, I put their names in alphabetical order (ignoring capital letters). This gives a consistent order when there is a tie.
High-value sales
I treated a high-value sale as one with revenue strictly above KES 50,000. A sale worth exactly KES 50,000 is not included because the code uses >.

I used a list comprehension in high_value() to select those sales. I then used a dictionary comprehension to count the qualifying sales for each store. The store count includes stores that had zero high-value sales.
Statistics
I wrote mean(), median() and minimum_maximum() rather than importing the statistics module. The statistics use the revenue of each individual transaction.
- The mean is the sum of the transaction revenues divided by the number of transactions.
- The median is the middle value after sorting the revenues. When there is an even number of values, it is the average of the two middle values.
- The minimum and maximum are found by going through the list with a loop. I did not use the built-in min() or max() functions in this function.
All three functions raise a ValueError when they are given an empty list.
Checking with pandas
The verify_with_pandas() function reads the original CSV again with pandas and applies the same cleaning rules. It then recalculates the total revenue, the revenue groupings, top three products, high-value counts and transaction statistics.

It prints MATCH when a result agrees with the result from my functions and MISMATCH when it does not. Small decimal differences are allowed when comparing calculated numbers because of floating-point precision.

If I get a mismatch, I would first check that both methods used the same valid rows, that the quantity and price cleaning rules are the same, and that the high-value condition is still strictly greater than KES 50,000. The program prints both values for a result that does not match so that the difference can be investigated.
Running the program
Install pandas for the verification step:
pip install pandas
Put sales_analysis.py and electronics_sales.csv in the same folder, then run:
python sales_analysis.py electronics_sales.csv
If the CSV has a different name or is in another folder, pass its path after the script name.