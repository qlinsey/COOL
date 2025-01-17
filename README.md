# COOL: a COhort OnLine analytical processing system

---

[Website](http://13.212.103.48:3001/) | [Documentation](http://13.212.103.48:3001/docs/tutorials/tutorial-csv) | [Blog](http://13.212.103.48:3001/blog) | [Demo](https://www.comp.nus.edu.sg/~dbsystem/cool/#/demo) | [GitHub](https://github.com/COOL-cohort/COOL)

---


## Introduction to COOL

![ COOL](./assets/img/p1.svg)

Different groups of people often have different behaviors or trends. For example, the bones of older people are more porous than those of younger people. It is of great value to explore the behaviors and trends of different groups of people, especially in healthcare, because we could adopt appropriate measures in time to avoid tragedy. The easiest way to do this is **cohort analysis**.

But with a variety of big data accumulated over the years, **query efficiency** becomes one of the problems that OnLine Analytical Processing (OLAP) systems meet, especially for cohort analysis. Therefore, COOL is introduced to solve the problems.

COOL is an online cohort analytical processing system that supports various types of data analytics, including cube query, iceberg query and cohort query.

With the support of several newly proposed operators on top of a sophisticated storage layer, COOL could provide high performance (near real-time) analytical response for emerging data warehouse domains.


## Key features of COOL

1. **Easy to use.** COOL is easy to deploy on local or on cloud via docker.
2. **Near Real-time Responses.** COOL is highly efficient, and therefore, can process cohort queries in near real-time analytical responses.
3. **Specialized Storage Layout.** A specialized storage layout is designed for fast query processing and reduced space consumption.
4. **Self-designed Semantics.** There are some novel self-designed semantics for the cohort query, which can simplify its complexity and improve its functionality.
5. **Flexible Integration.** Flexible integration with other data systems via common data formats(e.g., CSV, Parquet, Avro, and Arrow).
6. **Artificial Intelligence Model.** A new neural network model will be introduced soon.

## Quickstart

### BUILD

Simply run `mvn clean package`

### Required sources:

1. **dataset file**: a csv file with "," delimiter (normally dumped from a database table), and the table header is removed.
2. **dimension file**: a csv file with "," delimiter.
Each line of this file has two fields: the first field is the name of a column in the dataset, and the second field is a value of this column.
Each distinct value of each column in the dataset shall appear in this dimension file once.
3. **dataset schema file**: a `table.yaml` file specifying the dataset's columns and their measure fields.
4. **query file**: a yaml file specify the parameters for running query server.

### Load dataset

Before query processing, we need to load the dataset into COOL native format. The sample code to load csv dataset with data loader can be found in [CsvLoader.java](cool-core/src/main/java/com/nus/cool/functionality/CsvLoader.java).

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CsvLoader path/to/your/source/directory path/to/your/.yaml path/to/your/datafile path/to/output/datasource/directory
```

The five arguments in the command have the following meaning:
1. a unique dataset name given under the directory
2. the table.yaml (the third required source)
3. the dataset file (the first required source)
4. the output directory for the compacted dataset


### Execute queries

We provide an example for cohort query processing in [CohortAnalysis.java](cool-core/src/main/java/com/nus/cool/functionality/CohortAnalysis.java).

There are two types of queries in COOL. The first one includes two steps.

#### Cohort Query

- Select the specific users.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CohortSelection path/to/output/datasource/directory path/to/your/queryfile
```

- Executes cohort query users.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CohortAnalysis path/to/output/datasource/directory path/to/your/cohortqueryfile
```

- Executes the funnel query.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.FunnelAnalysis path/to/output/datasource/directory path/to/your/funnelqueryfile
```

#### OLAP Query

- Executes the following query in cool.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.IcebergLoader path/to/output/datasource/directory path/to/your/queryfile
```

### Example-Cohort Analysis

#### Load dataset

We have provided examples in `sogamo` directory and `health` directory. Now we take `sogamo` for example.

The COOL system supports CSV data format by default, and you can load `sogamo` dataset with the following command.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CsvLoader sogamo sogamo/table.yaml sogamo/test.csv datasetSource
```

In addition, you can run the following command to load dataset in other formats under the `sogamo` directory.

- parquet format data
```bash
$ java -jar cool-extensions/parquet-extensions/target/parquet-extensions-0.1-SNAPSHOT.jar sogamo sogamo/table.yaml sogamo/test.parquet datasetSource
```

- Arrow format data
```bash
$ java -jar cool-extensions/arrow-extensions/target/arrow-extensions-0.1-SNAPSHOT.jar sogamo sogamo/table.yaml sogamo/test.arrow datasetSource
```

- Avro format data
```bash
$ java -jar cool-extensions/avro-extensions/target/avro-extensions-0.1-SNAPSHOT.jar sogamo sogamo/table.yaml sogamo/avro/test.avro datasetSource sogamo/avro/schema.avsc
```

Finally, there will be a cube generated under the `datasetSource` directory, which is named `sogamo`.

#### Execute queries

We use the `health` dataset for example to demonstrate the cohort ananlysis.

- Select the specific users.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CohortSelection datasetSource health/query1-0.json
```

where the three arguments are as follows:
1. `datasetSource`: the output directory for the compacted dataset
2. `health`: the cube name of the compacted dataset
3. `health/query1-0.json`: the json file for the cohort query

- Display the selected all records of the cohort in terminal for exploration
```
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CohortExploration datasetSource health loyal
```

- Execute cohort query on the selected users.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CohortAnalysis datasetSource health/query1-1.json
```

- Execute cohort query on all the users.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CohortAnalysis datasetSource health/query2.json
```

Partial results for the query `health/query2.json` on the `health` dataset are as at [result2.json](health/result2.json)

We use the `sogamo` dataset for example to demonstrate the funnel analysis.

```bash
$ java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.FunnelAnalysis datasetSource sogamo/query1.json
```

### Example-OLAP Analysis

#### Load dataset

We have provided examples in `olap-tpch` directory.

The COOL system supports CSV data format by default, and you can load `tpc-h` dataset with the following command.

```bash
java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.CsvLoader tpc-h-10g olap-tpch/table.yaml olap-tpch/scripts/data.csv datasetSource
```

Finally, there will be a cube generated under the `datasetSource` directory, which is named `tpc-h-10g`.

#### Execute queries

Now we could execute OLAP query on the generated datasets. eg, if we want to run following query,

```sql
SELECT cout(*), sum(O_TOTALPRICE) 
FROM TPC_H WHERE O_ORDERPRIORITY = 2-HIGH AND R_NAME = EUROPE
GROUP BY N_NAME,R_NAME 
HAVING O_ORDERDATE >= '1993-01-01' AND O_ORDERDATE <= '1994-01-01' 
```

- Firstly define a query.json as shown in [query.json](olap-tpch/query.json)
- Then execute following cmd

```bash
java -cp ./cool-core/target/cool-core-0.1-SNAPSHOT.jar com.nus.cool.functionality.IcebergLoader datasetSource olap-tpch/query.json
```

 Results for the query  [query.json](olap-tpch/query.json) on the `tpc-h-10g` dataset are as at [result.json](olap-tpch/result2.json)

## HOW TO RUN WITH A SERVER

We can start the COOL's query server with the following command
```bash
$ java -jar cool-queryserver/target/cool-queryserver-0.1-SNAPSHOT.jar datasetSource 8080
```
where the argument is as follows:
1. `datasetSource`: the path to the repository of compacted datasets.
2. `8080`: the port of the server.

In this server, we implement many APIs and list their corresponding urls as follows:
- \[server:port]:v1
    - List all workable urls
- \[server:port]:v1/reload?cube=[cube_name]
    - Reload the cube
- \[server:port]:v1/list
    - List existing cubes
- \[server:port]:v1/cohort/list?cube=[cube_name]
    - List all cohorts from the selected cube
- \[server:port]:v1/cohort/selection
    - Cohort Selection
- \[server:port]:v1/cohort/exploration
  - Cohort Exploration
- \[server:port]:v1/cohort/analysis
    - Perform cohort analysis
- \[server:port]:v1/funnel/analysis
    - Perform funnel analysis
- \[server:port]:v1/olap/iceberg
    - Perform iceberg query

## CONNECT TO EXTERNAL STORAGE SERVICES
COOL has an [StorageService](cool-core/src/main/java/com/nus/cool/storageservice/StorageService.java) interface, which will allow COOL standalone server/workers (coming soon) to handle data movement between local and an external storage service. A sample implementation for HDFS connection can be found under the [hdfs-extensions](cool-extensions/hdfs-extensions/).


## Publication
* Z. Xie, H. Ying, C. Yue, M. Zhang, G. Chen, B. C. Ooi. [Cool: a COhort OnLine analytical processing system](https://www.comp.nus.edu.sg/~ooibc/icde20cool.pdf), in 2020 IEEE 36th International Conference on Data Engineering, pp.577-588, 2020.
* Q. Cai, Z. Xie, M. Zhang, G. Chen, H.V. Jagadish and B.C. Ooi. [Effective Temporal Dependence Discovery in Time Series Data](http://www.comp.nus.edu.sg/~ooibc/cohana18.pdf), in Proceedings of the VLDB Endowment, 11(8), pp.893-905, 2018.
* Z. Xie, Q. Cai, F. He, G.Y. Ooi, W. Huang, B.C. Ooi. [Cohort Analysis with Ease](https://dl.acm.org/doi/10.1145/3183713.3193540), in Proceedings of the 2018 International Conference on Management of Data, pp.1737-1740, 2018.
* D. Jiang, Q. Cai, G. Chen, H. V. Jagadish, B. C. Ooi, K.-L. Tan, and A. K. H. Tung. [Cohort Query Processing](http://www.vldb.org/pvldb/vol10/p1-ooi.pdf), in Proceedings of the VLDB Endowment, 10(1), 2016.
