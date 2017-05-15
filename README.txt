Group 18 - Assignment 3 - Record Manager | Version 1.0 | 11/10/2015





Description

---------------------------------------------------------


The record manager is used to handle table which can insert records, delete records, update records, and scan through the records.



How to run

-----------------------------------------------------------


1. Open terminal and Clone from BitBucket to the required location.

2. Navigate to assign3 folder.

3. Use make command to execute record manager, test_assign3_1,
	$ make all
4. Use make command to execute test expr, test_expr,
	$ make expr
5. To clean,
	$ make clean





BONUS Implementation - TOMBSTONE for deleting records
------------------------------------------------------
We have used A tombstone flag for each deleted record which will be prefixed to the actual record to be deleted.

For Eg. The actual record before deletion is stored as:	[7-0] (a:7,b:gggg,c:3)
	The actual record after deletion is stored as:	DELETED_RECORD[7-0] (a:7,b:gggg,c:3)

Solution Description

-----------------------------------------------------------



Serializer - The solution uses the already available rm_serializer.c for serializing the structures into string such as schemas, tables, records, etc. which can be used to store into the page file.

Deserializer - A new file rm_deserializer.c is created to deserialize the string into the required structures.

	Example - String format of schema with 3 attribyes and a key:

		Schema with 3 attributes (a:INT, b:FLOAT, c STRING[20]) with keys: (a)

RM_TableInfo - a structure which stores table information such as noOfTables, schemaSize etc.

RM_RecordMgmt - a structure used to store the storage information of records such as buffer pool info, free records info etc.

attrOffset - this function is used to retrieve the offset value of an attribute's position in a schema.

Additional error codes in dberror.h
-----------------------------------
RC_TABLE_ALREADY_EXISTS - 400
RC_RM_UPDATE_NOT_POSSIBLE_ON_DELETED_RECORD - 401
RC_RM_NO_DESERIALIZER_FOR_THIS_DATATYPE - 402


Table and Manager
------------------
initRecordManager (void *mgmtData)
	this function is used to initialize the record manager by allocating memory.

shutdownRecordManager ()
	this function is used to free all the memory associated with the structures used in record manager.

createTable (char *name, Schema *schema)
	this function is used to serialize the schema passed as a parameter "schema".
The serialized data is then written to the first postion of the page file.

openTable (RM_TableData *rel, char *name)
	this function opens the page file passed as a parameter "name" and opens it in the read-write mode.
The Buffer Pool is initialized based on FIFO strategy and the fist page is pinned from the page file.
The schema stored as a string in the page file is deserialized in the schema structure and stored in the RM_TableData structure.

closeTable (RM_TableData *rel)
	this function calls shutdownBufferPool and free all the memory allocated in the RM_TableData structure.

deleteTable (char *name)
	this function calls destroyPageFile function to delete the page file(table) as specified by "name" parameter.

getNumTuples (RM_TableData *rel)
	this function is used to get the count total number of tuples present in the table.


Records handling in a table
----------------------------
insertRecord (RM_TableData *rel, Record *record)
	this function is used to insert  a record passed as a parameter in the RM_TableData and assigns RID to the page.

deleteRecord (RM_TableData *rel, RID id)
	this function is used to delete a record as specified by RID from the RM_TableData.
A tombstone flag(DELETEd_RECORD) is prefixed to the record required to be deleted.

updateRecord (RM_TableData *rel, Record *record)
	this function is used to update the record in the RM_TableData.
If a tombstone flag(DELETEd_RECORD) is prefixed to the record the update operation will not execute.

getRecord (RM_TableData *rel, RID id, Record *record)
	this function is used to retrieve a record as specified by RID from the RM_TableData and store it in the "record" parameter.

Scans
-----
startScan (RM_TableData *rel, RM_ScanHandle *scan, Expr *cond)
	this function is used to initialize the RM_ScanHandle and RM_ScanMgmt structures and its attributes.

next (RM_ScanHandle *scan, Record *record)
	this function is used to search tuples based on the scan condition which is specified.
If scan condition is NULL then it returns all the tuples until the end the table.
If there is a valid scan condition then the tuple is evaluated according to the scan condition using EvalExpr() from expr.c
This will run until there no more tuples in the table.

closeScan (RM_ScanHandle *scan)
	this function indicates the record manager that all the resources can be cleaned up. All the memory allocated during scans are free.

Dealing with schemas
---------------------
getRecordSize (Schema *schema)
	this function returns the size of the record.

createSchema (int numAttr, char **attrNames, DataType *dataTypes, int *typeLength, int keySize, int *keys)
	this function is used to create a new schema and initailize all its values as specified by the parameters.

freeSchema (Schema *schema)
	this function frees up the memory allocated in a schema.


Dealing with records and attribute values
------------------------------------------
createRecord (Record **record, Schema *schema)
	this function is used to create a record.
Memory is allocated to the data of the record.

freeRecord (Record *record)
	this function frees up the memory allocated for a record.

getAttr (Record *record, Schema *schema, int attrNum, Value **value)
	this function is used to retrieve attribute values of a record.

setAttr (Record *record, Schema *schema, int attrNum, Value *value)
	this function is used to set attribute values of a record.




Test Cases

-----------------------------------------------------------

Files: test_assign3_1.c

1. The program verifies all the test cases that are mentioned in the test file i.e test_assign3_1 and ensures that there are no errors.

2. The program also verfies the test case for test_expr.c and ensure there are no errors. 

3. Both the test cases run perfectly. Some memory leaks in the test cases are resolved.





Team Members: - Group 18

-----------------------------------------------------------

Loy Mascarenhas

Pranitha Nagavelli

Shalin Chopra
