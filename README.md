# IteratorX: simplest iterator for IO

## 1. Reader: JdbcReader, FileReader

Readers can iteratively read data into json objects, including JdbcReader and FileReader.

### 1.1. JdbcReader: read jdbc table rows into json objects

Read each jdbc Table Row into JSONObject iteratively.

```	
	// create jdbc reader
	final JdbcReader jdbcReader = new JdbcReader(
			new JdbcDataSourceBuilder().setUrl("jdbc:postgresql://10.23.112.2:3333/dbname")
					.setUser("username").setPassword("password").build());
	
	// fetch by iterable
	for (final JSONObject item : jdbcReader.read("select * from tablename")) {
		System.err.println(item);
	}
	
	// fetch all into one collection
	final Collection<JSONObject> items = jdbcReader.readAll("select * from tablename where id = ?", "param");
	for (final JSONObject item : items) {
		System.err.println(item);
	}
```
### 1.2. FileReader: read file content lines into json objects
todo

## 2. Parallels: Threads, Flink, RxJava

As we always need to process data in parallel, we support many parallels engines: Threads(ThreadPool), Flink and RxJava.

All parallels engines are support both JSONObject and other parametized type.

### 2.1. Threads: using ThreadPool to process data in parallel

Fixed-size thread pool are used to process data in multi-threads, the default thread size is the 3 times of available processors.

```	
	// process each item parallelly using thread pool
	Threads.from(jdbcReader.read("select * from tablename")).forEach(item -> {
		System.err.println(item);
	});
	
	// process batch data parallelly
	Threads.from(jdbcReader.read("select * from tablename")).forBatch(items -> {
		for (final JSONObject item : items) {
			System.err.println(item);
		}
	});
```
### 2.2. Flink: using Flink to process data in parallel

Flink can run in both standalone local mode and remote cluster mode, this is fantastic to debug and execute. We prefer using Flink engine to process big data in parallel. The default parallelism is the 3 times of available processors.

```	
	// process each item parallelly using Flink engine
	Flink.from(jdbcReader.read("select * from tablename")).forEach(item -> {
		System.err.println(item);
	});
	
	// process batch data parallelly
	Flink.from(jdbcReader.read("select * from tablename")).forBatch(items -> {
		for (final JSONObject item : items) {
			System.err.println(item);
		}
	});
	
	// use DataSet directly to enable all Flink power
	Flink.from(jdbcReader.read("select * from tablename")).dataSet().distinct().count();
	
```
### 2.3. RxJava: using RxJava to process data in parallel

We also support RxJava engine, the default parallelism is the 3 times of available processors.

Known issues: the RxJava will not quit automatically when processing data finished, we will try to fix this bug.

```	
	// create jdbc reader
	final JdbcReader jdbcReader = new JdbcReader(
			new JdbcDataSourceBuilder().setUrl("jdbc:postgresql://10.23.112.2:3333/dbname")
					.setUser("username").setPassword("password").build());

	// process each item parallely using RxJava engine
	RxJava.from(jdbcReader.read("select * from tablename")).forEach(item -> {
		System.err.println(item);
	});
	
	// process batch data parallely
	RxJava.from(jdbcReader.read("select * from tablename")).forBatch(items -> {
		for (final JSONObject item : items) {
			System.err.println(item);
		}
	});
	
	// use Observable directly
	RxJava.from(jdbcReader.read("select * from tablename")).observable().distinct().count();
```


