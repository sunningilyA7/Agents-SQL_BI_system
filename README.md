# Agents-SQL_BI_system

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

   
>>> import langchain
>>> from langchain.utilities import SQLDatabase
>>> from langchain.utilities import SQLDatabase
>>> from langchain.llms import OpenAI
>>> from langchain_experimental.sql import SQLDatabaseChain

>>> db = SQLDatabase.from_uri("sqlite:///test.db")


>>> llm = OpenAI(temperature=0, verbose=True, openai_api_key  = '')

>>> db_chain = SQLDatabaseChain.from_llm(llm, db,  verbose=True)
>>> 
>>> response = db_chain.run("How many different kinds of dry goods are there?")
/Library/Frameworks/Python.framework/Versions/3.9/lib/python3.9/site-packages/langchain_core/_api/deprecation.py:119: LangChainDeprecationWarning: The method `Chain.run` was deprecated in langchain 0.1.0 and will be removed in 0.3.0. Use invoke instead.
  warn_deprecated(


> Entering new SQLDatabaseChain chain...
How many different kinds of dry goods are there?
SQLQuery:SELECT COUNT(DISTINCT Type) FROM DryGoods
SQLResult: [(1,)]
Answer:1
> Finished chain.
>>> print(response)
1
>>> response = db_chain.run("What is the average selling price？")


> Entering new SQLDatabaseChain chain...
What is the average selling price
SQLQuery:SELECT AVG(SalePrice) FROM DryGoods
SQLResult: [(28.76666666666667,)]
Answer:28.76666666666667
> Finished chain.
>>> print(response)
28.76666666666667
>>> 


