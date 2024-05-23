# Agents-SQL_BI_system

Summary: using agents has higher accuracy than without it 


1. connect sql (local)


import sqlite3
conn = sqlite3.connect('test.db')
cursor = conn.cursor()
cursor.execute('''
        CREATE TABLE DryGoods (
            ID INTEGER PRIMARY KEY, 
            Name TEXT NOT NULL, 
            Type TEXT NOT NULL, 
            Source TEXT NOT NULL, 
            PurchasePrice REAL, 
            SalePrice REAL
        );
    ''')

  dry_goods = [
    ('kelp', 'DryFood', 'liaoning', 5.2, 8.9),
    ('mushroom', 'DryFood', 'guangdong', 26.5, 33.2),
    ('black_fungus', 'DryFood', 'shandong', 34.6, 44.2)
]


for item in dry_goods:
    cursor.execute('''
        INSERT INTO DryGoods (Name, Type, Source, PurchasePrice, SalePrice) 
        VALUES (?, ?, ?, ?, ?);
    ''', item)

conn.commit()

conn.close()



2. connect langchain, also need openai key

   
import langchain
from langchain.utilities import SQLDatabase
from langchain.utilities import SQLDatabase
from langchain.llms import OpenAI
from langchain_experimental.sql import SQLDatabaseChain

db = SQLDatabase.from_uri("sqlite:///test.db")


llm = OpenAI(temperature=0, verbose=True, openai_api_key  = '')

db_chain = SQLDatabaseChain.from_llm(llm, db,  verbose=True)

response = db_chain.run("How many different kinds of dry goods are there?")
/Library/Frameworks/Python.framework/Versions/3.9/lib/python3.9/site-packages/langchain_core/_api/deprecation.py:119: LangChainDeprecationWarning: The method `Chain.run` was deprecated in langchain 0.1.0 and will be removed in 0.3.0. Use invoke instead.
  warn_deprecated(


Entering new SQLDatabaseChain chain...
How many different kinds of dry goods are there?
SQLQuery:SELECT COUNT(DISTINCT Type) FROM DryGoods
SQLResult: [(1,)]
Answer:1
Finished chain.
print(response)
1
response = db_chain.run("What is the average selling price？")


Entering new SQLDatabaseChain chain...
What is the average selling price
SQLQuery:SELECT AVG(SalePrice) FROM DryGoods
SQLResult: [(28.76666666666667,)]
Answer:28.76666666666667
Finished chain.
print(response)
28.76666666666667



3. using agents methods
from langchain.agents.agent_toolkits import SQLDatabaseToolkit
 from langchain.agents.agent_types import AgentType
agent_executor = create_sql_agent(
...     llm=llm,
...     toolkit=SQLDatabaseToolkit(db=db, llm=llm),
...     verbose=True,
...     agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
... )


questions = [ 'How many different kinds of dry goods are there?',' What is the average selling price?']
for question in questions:
...     response = agent_executor.run(question)
...     print(response)
... 


 Entering new SQL Agent Executor chain...
 I need to find a way to query the database for the different kinds of dry goods.
Action: sql_db_query_checker
Action Input: SELECT COUNT(DISTINCT kind) FROM dry_goods SELECT COUNT(DISTINCT kind) FROM dry_goods I need to use the sql_db_query tool to execute the query and get the result.
Action: sql_db_query
Action Input: SELECT COUNT(DISTINCT kind) FROM dry_goodsError: (sqlite3.OperationalError) no such table: dry_goods
[SQL: SELECT COUNT(DISTINCT kind) FROM dry_goods]
(Background on this error at: https://sqlalche.me/e/20/e3q8) I need to use the sql_db_list_tables tool to check if the table "dry_goods" exists in the database.
Action: sql_db_list_tables
Action Input: DryGoods I need to use the sql_db_schema tool to get the schema and sample rows for the "DryGoods" table.
Action: sql_db_schema
Action Input: DryGoods
CREATE TABLE "DryGoods" (
	"ID" INTEGER, 
	"Name" TEXT NOT NULL, 
	"Type" TEXT NOT NULL, 
	"Source" TEXT NOT NULL, 
	"PurchasePrice" REAL, 
	"SalePrice" REAL, 
	PRIMARY KEY ("ID")
)

/*
3 rows from DryGoods table:
ID	Name	Type	Source	PurchasePrice	SalePrice
1	kelp	DryFood	liaoning	5.2	8.9
2	mushroom	DryFood	guangdong	26.5	33.2
3	black_fungus	DryFood	shandong	34.6	44.2
*/ I now know the final answer is 3 different kinds of dry goods. 
Final Answer: 3

Finished chain.
3


Entering new SQL Agent Executor chain...
 We need to query the database to get the average selling price.
Action: sql_db_query
Action Input: SELECT AVG(selling_price) FROM salesError: (sqlite3.OperationalError) no such table: sales
[SQL: SELECT AVG(selling_price) FROM sales]
(Background on this error at: https://sqlalche.me/e/20/e3q8)We need to check the schema to see if there is a table called "sales".
Action: sql_db_schema
Action Input: salesError: table_names {'sales'} not found in database We need to check the list of tables to see if there is a table called "sales".
Action: sql_db_list_tables
Action Input: DryGoods We need to check the schema for the "DryGoods" table to see if there is a "selling_price" column.
Action: sql_db_schema
Action Input: DryGoods
CREATE TABLE "DryGoods" (
	"ID" INTEGER, 
	"Name" TEXT NOT NULL, 
	"Type" TEXT NOT NULL, 
	"Source" TEXT NOT NULL, 
	"PurchasePrice" REAL, 
	"SalePrice" REAL, 
	PRIMARY KEY ("ID")
)

/*
3 rows from DryGoods table:
ID	Name	Type	Source	PurchasePrice	SalePrice
1	kelp	DryFood	liaoning	5.2	8.9
2	mushroom	DryFood	guangdong	26.5	33.2
3	black_fungus	DryFood	shandong	34.6	44.2
*/ We need to rewrite our query to use the correct table and column names.
Action: sql_db_query
Action Input: SELECT AVG(SalePrice) FROM DryGoods[(28.76666666666667,)] We now have the average selling price.
Final Answer: The average selling price is $28.77.

Finished chain.
The average selling price is $28.77.



