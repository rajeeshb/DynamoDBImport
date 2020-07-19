## Introduction

DDBImport is a python script to load data from JSON file into DynamoDB table. 

Usage:

~~~~
python DDBImport.py -r <region_name> -t <table_name> -s <source_file> -p <process_count>
~~~~

Example:

~~~~
python DDBImport.py -r us-east-1 -t TestTable -s test.json -p 8
~~~~
  
The script launches multiple processes to do the work. The processes poll from a common queue for data to write. When the queue is empty, the processes continues to poll the queue for another 60 seconds to make sure it does not miss anything. 

It is safe to use 1 process per vCPU core. If you have an EC2 instance with 4 vCPU cores, it is OK to set the process count to 4. The BatchWriteItem API is used to perform the import. Depending on the size of the items, each process can consume approximately 1000 WCU during the import. 

Tested on an EC2 instance with the c3.8xlarge instance type. The data set contains 10,000,000 items, with each item being approximately 170 bytes. The size of the JSON file is 1.7 GB. The DynamoDB table has 40,000 provisioned WCU. Perform the import with 32 threads, and the import is completed in 7 minutes. The peak consumed WCU is approximately 32,000 (average value over a 1-minute period).

## Data Format

DDBImport accepts regular JSON data format. For example:

~~~~
[{"hash": "ABC", "range": "123", "val_1": "ABCD"},
{"hash": "BCD", "range": "234", "val_2": 1234},
{"hash": "CDE", "range": "345", "val_1": "ABCD", "val_2": 1234}]
~~~~

We also provide a data generation utility GenerateTestData.py for testing purposes. 

Usage:

~~~~
python GenerateTestData.py -c <item_count> -f <output_file>
~~~~
  
Example:

~~~~
python GenerateTestData.py -c 1000000 -f test.json
~~~~

Run this with the following command format:

~~~~
python DDBImport.py DynamoDB_Table_Name data.json region n_threads
~~~~

For example:

~~~~
python DDBImport.py DynamoDB_Table_Name data.json us-east-1 2
python DDBImport.py -r us-east-1 -t TestTable -s test.json -p 8
~~~~

