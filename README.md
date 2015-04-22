# gpudb
Automatically exported from code.google.com/p/gpudb
QuickStart  
Quick Start Guide. Updated Sep 23, 2013 by yuanyuan...@gmail.com
GPUDB Start Guide
Summary
System Requirement
Source Code
Write Schema File
Data Generation
Generate Data Loader
SSBM Plain Data Generation
Load SSBM Data
Data Compression
Code Generation
GPUDB Start Guide
Summary
This is a translator that translates a given SQL query into CUDA codes (executed on NVIDIA GPUs) or OpenCL codes (executed on devices that support OpenCL). You need to:

Define the schema and generate the raw text-based data.
Generate the data loader that transform the raw data into our supported data format.
(Optional) Compress the data to obtain better performance.
Translate the SQL query and execute the query.
The remaining contents in this guide will explain each step in details. All the paths used in the following content are relative to the top directory of the source code.
System Requirement
A Linux system with Python installed.
CUDA toolkit (4.0 or later), AMD APP SDK or Intel OpenCL SDK installed.
NVIDIA GPUs, AMD GPUs or CPUs that support OpenCL.
Source Code
You can check out the source codes here.

Write Schema File
Please refer to wiki page Schema to learn how to write schema file. An example schema file is located in test/ssb_test/ssb.schema.

Data Generation
Please refer to StorageFormat to get a general idea of the format of the data. You must ensure that the data are stored in the correct format before running the queries. We provide a tool that can transform the plain text data (columns must be separated by '|') into our supported storage format.

Generate Data Loader
Execute

./translate.py your_schema_file
to generate a C file that can be used to transform the raw text data into our supported data format. The C file is generated in src/utility/load.c.

For SSBM queries, execute the following command:

./translate.py test/ssb_test/ssb.schema
The following shows how to transform the original SSBM data into our supported storage format.

SSBM Plain Data Generation
The dbgen to generate original SSBM data is located in test/dbgen directory. You need to generate the plain text data. The following steps show how to generate the data with a scale factor of 1.

Enter test/dbgen directory, and execute "make"
Execute "./dbgen -vfF -T a -s 1" to generate data with scale factor 1
Load SSBM Data
The original SSBM data cannot be directly used by our engine. The data need to be transformed into column-stored data. The following steps show how to transform the original SSBM data.

Enter src/utility directory and execute "make loader", which will generate "gpuDBLoader" to transform the original data.
To load data for the previously generated SSBM data, execute the following command: ./gpuDBLoader --lineorder ../../test/dbgen/lineorder.tbl --ddate ../../test/dbgen/date.tbl --customer ../../test/dbgen/customer.tbl --supplier ../../test/dbgen/supplier.tbl --part ../../test/dbgen/part.tbl
Generally speaking, gpuDBLoader accepts arguments in the format "--table_name table_file". Each table column is stored in a disk file whose name is composed of the table name in upper case followed by the index of the column in the table. Users can specify where the data will be generated by using "--datadir dir". The data will be generated in the current directory by default.

Data Compression
Data can be compressed to get a better query performance. You can skip to the next section if you don't need to compress the data.

Run Length Encoding
Sort. To get a better compression ratio, the column that is compressed using RLE scheme should be sorted first. To sort the column, users can either use the "sort" command to sort the plain text SSBM data on particular columns before transforming the data into column-stored formant, or they can enter src/utility directory and execute "make sort" to generate a file named "columnSort" that will sort a table on a particular integer column.
Usage: ./columnSort inputPrefix outputPrefix index. inputPrefix is the name of the table to be sorted, outputPrefix is the name of the generated sorted table and index specifies the sorted column.
Compression. Enter src/utility and execute "make rle" to generate the file "rleCompression" to compress the column using RLE scheme.
Usage: ./rleCompression inputColumn outputColumn. inputColumn specifies the name of the sorted column to be compressed using RLE, and outputColumn is the name of the generated compressed column.
Dictionary Encoding
Enter src/utility and execute "make dict" to generate the file "dictCompression" to compress the column using dictionary encoding scheme.
Usage: ./dictCompression inputColumn outputColumn. inputColumn specifies the name of the column to be compressed using dictionary encoding scheme, and outputColumn is the name of the generated compressed column.
Code Generation
Usage: ./translate.py query-file.sql schema-file.schema

query-file.sql should contain the SQL query you want to translate. Currently the suppported SQL queries are listed in test/ssb_test directory
schema-file.schema is the schema file that describes the data. Currently the schema file for star schema benchmark is in test/ssb_test/ssb.schema.
After executing the aforementioned command, enter src/cuda or src/opencl directory and execute "Make gpudb", which will generate an executable file named "GPUDATABASE". Executing "./GPUDATABASE --datadir dir" will run the query on GPU. The "datadir" argument specifies where the data are stored.
