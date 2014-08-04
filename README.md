![thumbnail](./images/uniVocity-parsers.png)

Welcome to uniVocity-parsers
============================

uniVocity-parsers is a collection of extremely fast and reliable parsers for Java. It provides a consistent interface for handling different file formats,
and a solid framework for the development of new parsers.


## Quick Overview ##
The project was started and coded by [uniVocity Software](http://www.univocity.com), an Australian company that develops 
[uniVocity](http://www.univocity.com), a commercial data integration API for Java.

It soon became apparent that many parsers out there didn't provide enough flexibility, throughput or reliability for massive and diverse (a nice word for messy) inputs.
Another inconvenience was the difficulty in extending these parsers and dealing with a different beast for each format.          

We decided to then build our own architecture for parsing text files from the ground up.
The main goal of this architecture is to provide maximum performance and flexibility while making it easy for anyone to create new parsers.

## Parsers ##
uniVocity-parsers currently provides parsers for:

- CSV files (it's the fastest CSV parser for Java you can find)

- Fixed-width files

We will introduce more parsers over time. Note many delimiter-separated formats, such as pipe-separated, are subsets of CSV and our CSV parser should handle them.
We are planning to introduce parsers for this and other specific formats to uniVocity-parsers later on.
Please let us know what you need the most by sending and e-mail to `parsers@univocity.com`.
We will introduce parsers for formats that are of public interest.      
 
We also documented every single class for you, so you can try to create your own parsers for your own particular purposes. 
We will help anyone building their own parsers, and offer commercial support for all parsers included in the API (send us an e-mail to `support@univocity.com`, 
a dedicated team of experts are ready to assist you).

## Installation ##

Just download the jar file from [here](http://central.maven.org/maven2/com/univocity/univocity-parsers/1.0.0/univocity-parsers-1.0.0.jar). 

Or, if you use maven, simply add the following to your `pom.xml`

```xml

...
<dependency>
	<groupId>com.univocity</groupId>
	<artifactId>univocity-parsers</artifactId>
	<version>1.0.0</version>
	<type>jar</type>
</dependency>
...

```

## Background ##
uniVocity-parsers have the following functional requirements:

1. Support parsing and writing of text files in tabular format, especially:

	1.1 CSV files 
	
	1.2 Fixed-width files
	
2. Handle common non-standard functions such as

	2.1 File comments
	
	2.2 Partial reads
	
	2.3 Record skipping
	
3. Column selection

4. Annotation based mapping with data conversions

5. Handle edge cases such as multi-line fields and portable newlines  

6. Process the input in parallel.

And these non-functional requirements:

1. Be fast and flexible.

1. Have no external dependencies to existing libraries.

2. Be simple to use.

3. Provide a consistent API for different parsers.

4. Be flexible and heavily configurable.

5. Be extremely fast and memory efficient - yes, we micro optimize.  

6. Provide an extensible architecture: You should be able to write your own parser using ~200 lines of code and have all of the above for free.


## Examples ##

### Reading

In the following examples, the [example file](./src/test/resources/examples/example.csv) will be used as the input. It is not as simple as you might think. 
We've seen some known CSV parsers being unable to read this one correctly:


```

	
# This example was extracted from Wikipedia (en.wikipedia.org/wiki/Comma-separated_values)
#
# 2 double quotes ("") are used as the escape sequence for quoted fields, as per the RFC4180 standard
#  

Year,Make,Model,Description,Price
1997,Ford,E350,"ac, abs, moon",3000.00
1999,Chevy,"Venture ""Extended Edition""","",4900.00
   
# Look, a multi line value. And blank rows around it!
     
1996,Jeep,Grand Cherokee,"MUST SELL!
air, moon roof, loaded",4799.00
1999,Chevy,"Venture ""Extended Edition, Very Large""",,5000.00
,,"Venture ""Extended Edition""","",4900.00



```

#### To read all rows of a CSV (the quick and easy way).


```java

	
	
	CsvParserSettings settings = new CsvParserSettings();
	//the file used in the example uses '\n' as the line separator sequence.
	//the line separator sequence is defined here to ensure systems such as MacOS and Windows
	//are able to process this file correctly (MacOS uses '\r'; and Windows uses '\r\n').
	settings.getFormat().setLineSeparator("\n");
	
	// creates a CSV parser
	CsvParser parser = new CsvParser(settings);
	
	// parses all rows in one go.
	List<String[]> allRows = parser.parseAll(getReader("/examples/example.csv"));
	
	


```

The output will be:


```

	1 [Year, Make, Model, Description, Price]
	-----------------------
	2 [1997, Ford, E350, ac, abs, moon, 3000.00]
	-----------------------
	3 [1999, Chevy, Venture "Extended Edition", null, 4900.00]
	-----------------------
	4 [1996, Jeep, Grand Cherokee, MUST SELL!
	air, moon roof, loaded, 4799.00]
	-----------------------
	5 [1999, Chevy, Venture "Extended Edition, Very Large", null, 5000.00]
	-----------------------
	6 [null, null, Venture "Extended Edition", null, 4900.00]
	-----------------------


```

#### To read all rows of a CSV (iterator-style).


```java

	
	
	// creates a CSV parser
	CsvParser parser = new CsvParser(settings);
	
	// call beginParsing to read records one by one, iterator-style.
	parser.beginParsing(getReader("/examples/example.csv"));
	
	String[] row;
	while ((row = parser.parseNext()) != null) {
		println(out, Arrays.toString(row));
	}
	
	// The resources are closed automatically when the end of the input is reached,
	// or when an error happens, but you can call stopParsing() at any time.
	
	// You only need to use this if you are not parsing the entire content.
	// But it doesn't hurt if you call it anyway.
	parser.stopParsing();
	
	


```

#### Read all rows of a CSV (the powerful version).

To have greater control over the parsing process, use a [RowProcessor](./src/main/java/com/univocity/parsers/common/processor/RowProcessor.java). uniVocity-parsers provides some useful default implementations but you can always provide your own.

The following example uses [RowListProcessor](./src/main/java/com/univocity/parsers/common/processor/RowListProcessor.java), which just stores the rows read from a file into a List:


```java

	
	
	// The settings object provides many configuration options
	CsvParserSettings parserSettings = new CsvParserSettings();
	parserSettings.getFormat().setLineSeparator("\n");
	
	// A RowListProcessor stores each parsed row in a List.
	RowListProcessor rowProcessor = new RowListProcessor();
	
	// You can configure the parser to use a RowProcessor to process the values of each parsed row.
	// You will find more RowProcessors in the 'com.univocity.parsers.common.processor' package, but you can also create your own.
	parserSettings.setRowProcessor(rowProcessor);
	
	// Let's consider the first parsed row as the headers of each column in the file.
	parserSettings.setHeaderExtractionEnabled(true);
	
	// creates a parser instance with the given settings
	CsvParser parser = new CsvParser(parserSettings);
	
	// the 'parse' method will parse the file and delegate each parsed row to the RowProcessor you defined
	parser.parse(getReader("/examples/example.csv"));
	
	// get the parsed records from the RowListProcessor here.
	// Note that different implementations of RowProcessor will provide different sets of functionalities.
	String[] headers = rowProcessor.getHeaders();
	List<String[]> rows = rowProcessor.getRows();
	
	


```

Each row will contain: 


```

	[Year, Make, Model, Description, Price]
	=======================
	1 [1997, Ford, E350, ac, abs, moon, 3000.00]
	-----------------------
	2 [1999, Chevy, Venture "Extended Edition", null, 4900.00]
	-----------------------
	3 [1996, Jeep, Grand Cherokee, MUST SELL!
	air, moon roof, loaded, 4799.00]
	-----------------------
	4 [1999, Chevy, Venture "Extended Edition, Very Large", null, 5000.00]
	-----------------------
	5 [null, null, Venture "Extended Edition", null, 4900.00]
	-----------------------


```

You can also use a [ObjectRowProcessor](./src/main/java/com/univocity/parsers/common/processor/ObjectRowProcessor.java), which will produce rows of objects. You can convert values using an implementation of the [Conversion](./src/main/java/com/univocity/parsers/conversions/Conversion.java) interface.
The [Conversions](./src/main/java/com/univocity/parsers/conversions/Conversions.java) class provides some useful defaults for you.
For convenience, the [ObjectRowListProcessor](./src/main/java/com/univocity/parsers/common/processor/ObjectRowListProcessor.java) can be used to store all rows into a list. 


```java

	
	
	// ObjectRowProcessor converts the parsed values and gives you the resulting row.
	ObjectRowProcessor rowProcessor = new ObjectRowProcessor() {
		@Override
		public void rowProcessed(Object[] row, ParsingContext context) {
			//here is the row. Let's just print it.
			println(out, Arrays.toString(row));
		}
	};
	
	// converts values in the "Price" column (index 4) to BigDecimal
	rowProcessor.convertIndexes(Conversions.toBigDecimal()).set(4);
	
	// converts the values in columns "Make, Model and Description" to lower case, and sets the value "chevy" to null.
	rowProcessor.convertFields(Conversions.toLowerCase(), Conversions.toNull("chevy")).set("Make", "Model", "Description");
	
	// converts the values at index 0 (year) to BigInteger. Nulls are converted to BigInteger.ZERO.
	rowProcessor.convertFields(new BigIntegerConversion(BigInteger.ZERO, "0")).set("year");
	
	CsvParserSettings parserSettings = new CsvParserSettings();
	parserSettings.getFormat().setLineSeparator("\n");
	parserSettings.setRowProcessor(rowProcessor);
	parserSettings.setHeaderExtractionEnabled(true);
	
	CsvParser parser = new CsvParser(parserSettings);
	
	//the rowProcessor will be executed here.
	parser.parse(getReader("/examples/example.csv"));
	
	


```

After applying the conversions, the output will be:


```

	[1997, ford, e350, ac, abs, moon, 3000.00]
	[1999, null, venture "extended edition", null, 4900.00]
	[1996, jeep, grand cherokee, must sell!
	air, moon roof, loaded, 4799.00]
	[1999, null, venture "extended edition, very large", null, 5000.00]
	[0, null, venture "extended edition", null, 4900.00]


```

#### Using annotations to map your java beans: ####

Use the [Parsed](./src/main/java/com/univocity/parsers/annotations/Parsed.java) annotation to map the property to a field in the CSV file. You can map the property using a field name as declared in the headers,
or the column index in the input.

Each annotated operation maps to a [Conversion](./src/main/java/com/univocity/parsers/conversions/Conversion.java) and they are executed in the same sequence they are declared. 

This example works with [this csv file](./src/test/resources/examples/bean_test.csv)


```java

	class TestBean {
	
	// if the value parsed in the quantity column is "?" or "-", it will be replaced by null.
	@NullString(nulls = { "?", "-" })
	// if a value resolves to null, it will be converted to the String "0".
	@Parsed(defaultNullRead = "0")
	private Integer quantity;   // The attribute type defines which conversion will be executed when processing the value.
	// In this case, IntegerConversion will be used.
	// The attribute name will be matched against the column header in the file automatically.
	
	@Trim
	@LowerCase
	// the value for the comments attribute is in the column at index 4 (0 is the first column, so this means fifth column in the file)
	@Parsed(index = 4)
	private String comments;
	
	// you can also explicitly give the name of a column in the file.
	@Parsed(field = "amount")
	private BigDecimal amount;
	
	@Trim
	@LowerCase
	// values "no", "n" and "null" will be converted to false; values "yes" and "y" will be converted to true
	@BooleanString(falseStrings = { "no", "n", "null" }, trueStrings = { "yes", "y" })
	@Parsed
	private Boolean pending;
	
	//


```

Instances of annotated classes are created with by [BeanProcessor](./src/main/java/com/univocity/parsers/common/processor/BeanProcessor.java) and [BeanListProcessor](./src/main/java/com/univocity/parsers/common/processor/BeanListProcessor.java):


```java

	
	// BeanListProcessor converts each parsed row to an instance of a given class, then stores each instance into a list.
	BeanListProcessor<TestBean> rowProcessor = new BeanListProcessor<TestBean>(TestBean.class);
	
	CsvParserSettings parserSettings = new CsvParserSettings();
	parserSettings.setRowProcessor(rowProcessor);
	parserSettings.setHeaderExtractionEnabled(true);
	
	CsvParser parser = new CsvParser(parserSettings);
	parser.parse(getReader("/examples/bean_test.csv"));
	
	// The BeanListProcessor provides a list of objects extracted from the input.
	List<TestBean> beans = rowProcessor.getBeans();
	
	


```

Here is the output produced by the `toString()` method of each [TestBean](./src/test/java/com/univocity/parsers/examples/TestBean.java) instance:


```

	[TestBean [quantity=1, comments=?, amount=555.999, pending=true], TestBean [quantity=0, comments=" something ", amount=null, pending=false]]


```

#### Reading master-detail style files: ####

Use [MasterDetailProcessor](./src/main/java/com/univocity/parsers/common/processor/MasterDetailProcessor.java) or [MasterDetailListProcessor](./src/main/java/com/univocity/parsers/common/processor/MasterDetailListProcessor.java) to produce [MasterDetailRecord](./src/main/java/com/univocity/parsers/common/processor/MasterDetailRecord.java) objects.
A simple example a master-detail file is in [the master_detail.csv file](./src/test/resources/examples/master_detail.csv). 

Each [MasterDetailRecord](./src/main/java/com/univocity/parsers/common/processor/MasterDetailRecord.java) holds a master record row and its list of associated detail rows.


```java

	
	// 1st, Create a RowProcessor to process all "detail" elements
	ObjectRowListProcessor detailProcessor = new ObjectRowListProcessor();
	
	// converts values at in the "Amount" column (position 1 in the file) to integer.
	detailProcessor.convertIndexes(Conversions.toInteger()).set(1);
	
	// 2nd, Create MasterDetailProcessor to identify whether or not a row is the master row.
	// the row placement argument indicates whether the master detail row occurs before or after a sequence of "detail" rows.
	MasterDetailListProcessor masterRowProcessor = new MasterDetailListProcessor(RowPlacement.BOTTOM, detailProcessor) {
		@Override
		protected boolean isMasterRecord(String[] row, ParsingContext context) {
			//Returns true if the parsed row is the master row.
			//In this example, rows that have "Total" in the first column are master rows.
			return "Total".equals(row[0]);
		}
	};
	// We want our master rows to store BigIntegers in the "Amount" column
	masterRowProcessor.convertIndexes(Conversions.toBigInteger()).set(1);
	
	CsvParserSettings parserSettings = new CsvParserSettings();
	parserSettings.setHeaderExtractionEnabled(true);
	
	// Set the RowProcessor to the masterRowProcessor.
	parserSettings.setRowProcessor(masterRowProcessor);
	
	CsvParser parser = new CsvParser(parserSettings);
	parser.parse(getReader("/examples/master_detail.csv"));
	
	// Here we get the MasterDetailRecord elements.
	List<MasterDetailRecord> rows = masterRowProcessor.getRecords();
	MasterDetailRecord masterRecord = rows.get(0);
	
	// The master record has one master row and multiple detail rows.
	Object[] masterRow = masterRecord.getMasterRow();
	List<Object[]> detailRows = masterRecord.getDetailRows();
	


```

After printing the master row and its details rows, the output is:


```

	[Total, 100]
	=======================
	1 [Item1, 50]
	-----------------------
	2 [Item2, 40]
	-----------------------
	3 [Item3, 10]
	-----------------------


```

### Parsing fixed-width files (and other parsers to come)

All functionalities you have with the CSV file format are available for the fixed-width format (and any other parser we introduce in the future).

In the [example fixed-width file](./src/test/resources/examples/example.txt) we chose to fill the unwritten spaces with underscores ('_'), 
so in the parser settings we set the padding to underscore: 


```

	
	YearMake_Model___________________________________Description_____________________________Price___
	1997Ford_E350____________________________________ac, abs, moon___________________________3000.00_
	1999ChevyVenture "Extended Edition"______________________________________________________4900.00_
	1996Jeep_Grand Cherokee__________________________MUST SELL!
	air, moon roof, loaded_______4799.00_
	1999ChevyVenture "Extended Edition, Very Large"__________________________________________5000.00_
	_________Venture "Extended Edition"______________________________________________________4900.00_


```

The only thing you need to do is to instantiate a different parser:
 

```java

	
	
	// creates the sequence of field lengths in the file to be parsed
	FixedWidthFieldLengths lengths = new FixedWidthFieldLengths(4, 5, 40, 40, 8);
	
	// creates the default settings for a fixed width parser
	FixedWidthParserSettings settings = new FixedWidthParserSettings(lengths);
	
	//sets the character used for padding unwritten spaces in the file
	settings.getFormat().setPadding('_');
	
	//the file used in the example uses '\n' as the line separator sequence.
	//the line separator sequence is defined here to ensure systems such as MacOS and Windows
	//are able to process this file correctly (MacOS uses '\r'; and Windows uses '\r\n').
	settings.getFormat().setLineSeparator("\n");
	
	// creates a fixed-width parser with the given settings
	FixedWidthParser parser = new FixedWidthParser(settings);
	
	// parses all rows in one go.
	List<String[]> allRows = parser.parseAll(getReader("/examples/example.txt"));
	
	


```
 
Use [FixedWidthFieldLengths](./src/main/java/com/univocity/parsers/fixed/FixedWidthFieldLengths.java) to define what is the length of each field in the input. With that information we can then create the  [FixedWidthParserSettings](./src/main/java/com/univocity/parsers/fixed/FixedWidthParserSettings.java). 

The output will be: 

```

	1 [Year, Make, Model, Description, Price]
	-----------------------
	2 [1997, Ford, E350, ac, abs, moon, 3000.00]
	-----------------------
	3 [1999, Chevy, Venture "Extended Edition", null, 4900.00]
	-----------------------
	4 [1996, Jeep, Grand Cherokee, MUST SELL!
	air, moon roof, loaded, 4799.00]
	-----------------------
	5 [1999, Chevy, Venture "Extended Edition, Very Large", null, 5000.00]
	-----------------------
	6 [null, null, Venture "Extended Edition", null, 4900.00]
	-----------------------


```

All the rest is the same as with CSV parsers. You can use all [RowProcessor](./src/main/java/com/univocity/parsers/common/processor/RowProcessor.java)s for annotations, conversions, master-detail records 
and anything else we (or you) might introduce in the future.
 
We created a set of examples using fixed with parsing [here](./src/test/java/com/univocity/parsers/examples/FixedWidthParserExamples.java)


#### Column selection ####

Parsing the entire content of each record in a file is a waste of CPU and memory when you are not interested in all columns.
uniVocity-parsers lets you choose the columns you need, so values you don't want are simply bypassed.

The following examples can be found in the example class [SettingsExamples](./src/test/java/com/univocity/parsers/examples/SettingsExamples.java):

Consider the [example.csv](./src/test/resources/examples/example.csv) file with:


```

	
	Year,Make,Model,Description,Price
	1997,Ford,E350,"ac, abs, moon",3000.00
	1999,Chevy,"Venture ""Extended Edition""","",4900.00
	
	...


```

And the following selection:


```java

	
	// Here we select only the columns "Price", "Year" and "Make".
	// The parser just skips the other fields
	parserSettings.selectFields("Price", "Year", "Make");
	
	// let's parse with these settings and print the parsed rows.
	List<String[]> parsedRows = parseWithSettings(parserSettings);
	


```

The output will be:


```

	1 [3000.00, 1997, Ford]
	-----------------------
	2 [4900.00, 1999, Chevy]
	-----------------------
	...


```

The same output will be obtained with index-based selection.


```java

	
	// Here we select only the columns by their indexes.
	// The parser just skips the values in other columns
	parserSettings.selectIndexes(4, 0, 1);
	
	// let's parse with these settings and print the parsed rows.
	List<String[]> parsedRows = parseWithSettings(parserSettings);
	


```

You can also opt to keep the original row format with all columns, but only the values you are interested in being processed:


```java

	
	// Here we select only the columns "Price", "Year" and "Make".
	// The parser just skips the other fields
	parserSettings.selectFields("Price", "Year", "Make");
	
	// Column reordering is enabled by default. When you disable it,
	// all columns will be produced in the order they are defined in the file.
	// Fields that were not selected will be null, as they are not processed by the parser
	parserSettings.setColumnReorderingEnabled(false);
	
	// Let's parse with these settings and print the parsed rows.
	List<String[]> parsedRows = parseWithSettings(parserSettings);
	


```

Now the output will be:


```

	1 [1997, Ford, null, null, 3000.00]
	-----------------------
	2 [1999, Chevy, null, null, 4900.00]
	-----------------------
	3 [1996, Jeep, null, null, 4799.00]
	...


```

### Settings ###

Each parser has its own settings class, but many configuration options are common across all parsers. The following snippet demonstrates how to use each one of them: 


```java

	
	// sets what is the default value to use when the parsed value is null
	parserSettings.setNullValue("<NULL>");
	
	// sets what is the default value to use when the parsed value is empty
	parserSettings.setEmptyValue("<EMPTY>"); // for CSV only
	
	// sets the headers of the parsed file. If the headers are set then 'setHeaderExtractionEnabled(true)'
	// will make the parser simply ignore the first input row.
	parserSettings.setHeaders("a", "b", "c", "d", "e");
	
	// prints the columns in reverse order.
	// NOTE: when fields are selected, all rows produced will have the exact same number of columns
	parserSettings.selectFields("e", "d", "c", "b", "a");
	
	// does not skip leading whitespaces
	parserSettings.setIgnoreLeadingWhitespaces(false);
	
	// does not skip trailing whitespaces
	parserSettings.setIgnoreTrailingWhitespaces(false);
	
	// reads a fixed number of records then stop and close any resources
	parserSettings.setNumberOfRecordsToRead(9);
	
	// does not skip empty lines
	parserSettings.setSkipEmptyLines(false);
	
	// sets the maximum number of characters to read in each column.
	// The default is 4096 characters. You need this to avoid OutOfMemoryErrors in case a file
	// does not have a valid format. In such cases the parser might just keep reading from the input
	// until its end or the memory is exhausted. This sets a limit which avoids unwanted JVM crashes.
	parserSettings.setMaxCharsPerColumn(100);
	
	// for the same reasons as above, this sets a hard limit on how many columns an input row can have.
	// The default is 512.
	parserSettings.setMaxColumns(10);
	
	// Sets the number of characters held by the parser's buffer at any given time.
	parserSettings.setInputBufferSize(1000);
	
	// Disables the separate thread that loads the input buffer. By default, the input is going to be loaded incrementally
	// on a separate thread if the available processor number is greater than 1. Leave this enabled to get better performance
	// when parsing big files (> 100 Mb).
	parserSettings.setReadInputOnSeparateThread(false);
	
	// let's parse with these settings and print the parsed rows.
	List<String[]> parsedRows = parseWithSettings(parserSettings);
	


```

The output of the CSV parser with all these settings will be:


```

	1 [<NULL>, <NULL>, <NULL>, <NULL>, <NULL>]
	-----------------------
	2 [Price, Description, Model, Make, Year]
	-----------------------
	3 [3000.00, ac, abs, moon, E350, Ford, 1997]
	-----------------------
	4 [4900.00, <EMPTY>, Venture "Extended Edition", Chevy, 1999]
	-----------------------
	5 [<NULL>, <NULL>, <NULL>, <NULL>,    ]
	-----------------------
	6 [<NULL>, <NULL>, <NULL>, <NULL>,      ]
	-----------------------
	7 [4799.00, MUST SELL!
	air, moon roof, loaded, Grand Cherokee, Jeep, 1996]
	-----------------------
	8 [5000.00, <NULL>, Venture "Extended Edition, Very Large", Chevy, 1999]
	-----------------------
	9 [4900.00, <EMPTY>, Venture "Extended Edition", <NULL>, <NULL>]
	-----------------------
	...


```

#### Fixed-width settings ####


```java

	
	// For the sake of the example, we will not read the last 8 characters (for the Year column).
	// We will also NOT set the padding character to '_' so the output makes more sense for reading
	// and you can see what characters are being processed
	FixedWidthParserSettings parserSettings = new FixedWidthParserSettings(new FixedWidthFieldLengths(4, 5, 40, 40 /*, 8*/));
	
	//the file used in the example uses '\n' as the line separator sequence.
	//the line separator sequence is defined here to ensure systems such as MacOS and Windows
	//are able to process this file correctly (MacOS uses '\r'; and Windows uses '\r\n').
	parserSettings.getFormat().setLineSeparator("\n");
	
	// The fixed width parser settings has most of the settings for CSV.
	// These are the only extra settings you need:
	
	// If a row has more characters than what is defined, skip them until the end of the line.
	parserSettings.setSkipTrailingCharsUntilNewline(true);
	
	// If a record has less characters than what is expected and a new line is found,
	// this record is considered parsed. Data in the next row will be parsed as a new record.
	parserSettings.setRecordEndsOnNewline(true);
	
	RowListProcessor rowProcessor = new RowListProcessor();
	
	parserSettings.setRowProcessor(rowProcessor);
	parserSettings.setHeaderExtractionEnabled(true);
	
	FixedWidthParser parser = new FixedWidthParser(parserSettings);
	parser.parse(getReader("/examples/example.txt"));
	
	List<String[]> rows = rowProcessor.getRows();
	


```

The parser output with such configuration for parsing the [example.txt](./src/test/resources/examples/example.txt) file will be:


```

	1 [1997, Ford_, E350____________________________________, ac, abs, moon___________________________]
	-----------------------
	2 [1999, Chevy, Venture "Extended Edition"______________, ________________________________________]
	-----------------------
	3 [1996, Jeep_, Grand Cherokee__________________________, MUST SELL!]
	-----------------------
	4 [air,, moon, roof, loaded_______4799.00_]
	-----------------------
	5 [1999, Chevy, Venture "Extended Edition, Very Large"__, ________________________________________]
	-----------------------
	6 [____, _____, Venture "Extended Edition"______________, ________________________________________]
	-----------------------


```

As `recordEndsOnNewline = true `, lines 3 and 4 are considered different records, instead of a single, multi-line record.
For clarity: in line 4, the value of the *first column* is 'air,', the *second column* has value 'moon', and the *third* is 'roof, loaded_______4799.00_'.

### Format Settings

All parser settings have a default format definition. The following attributes are set by default for all parsers:

* `lineSeparator` (default *System.getProperty("line.separator");*): this is an array of 1 or 2 characters with the sequence that indicates the end of a line. 
	Using this, you should be able to handle files produced by different operating systems.
	Of course, if you want your line separator to be "#$", you can.    
	
* `normalizedNewline` (default *\n*): used to represent the sequence of 2 characters used as a line separator (e.g. *\r\n* in Windows). 
It is used by our parsers/writers to easily handle portable line separators.
 
  * When parsing, if the sequence of characters defined in *lineSeparator* is found while reading from the input, 
	it will be transparently replaced by the *normalizedNewline* character.
	 
  * When writing, *normalizedNewline* is replaced by the *lineSeparator* sequence. 
	
* `comment` (default *#*): if the first character of a line of text matches the comment character, then the row will be
	considered a comment and discarded from the input.  
	

#### CSV format ####


* `delimiter` (default *,*): value used to separate individual fields in the input.

* `quote` (default *"*): value used for escaping values where the field delimiter is part of the value (e.g. the value " a , b " is parsed as ` a , b `). 

* `quoteEscape` (default *"*): value used for escaping the quote character inside an already escaped value (e.g. the value " "" a , b "" " is parsed as ` " a , b " `).


#### Fixed width format ####

In addition to the default format definition, the fixed with format contains:

* `padding` (default *' '*): value used for filling unwritten spaces.

### Writing

As you can see in [WriterExamples.java](./src/test/java/com/univocity/parsers/examples/WriterExamples.java), writing is quite straightforward. All you need is an 
instance of java.io.Writer (to write the values you provide to some output resource) and a settings object with the configuration of how the values should be written.

#### Quick and simple writing example ####

You can write your data in CSV format using just 3 lines of code:


```java

	
	
	// All you need is to create an instance of CsvWriter with the default CsvWriterSettings.
	// By default, only values that contain a field separator are enclosed within quotes.
	// If quotes are part of the value, they are escaped automatically as well.
	// Empty rows are discarded automatically.
	CsvWriter writer = new CsvWriter(outputWriter, new CsvWriterSettings());
	
	// Write the record headers of this file
	writer.writeHeaders("Year", "Make", "Model", "Description", "Price");
	
	// Here we just tell the writer to write everything and close the given output Writer instance.
	writer.writeRowsAndClose(rows);
	
	


```

This will produce the following output:


```

	Year,Make,Model,Description,Price
	1997,Ford,E350,"ac, abs, moon",3000.00
	1999,Chevy,Venture "Extended Edition",,4900.00
	1996,Jeep,Grand Cherokee,"MUST SELL!
	air, moon roof, loaded",4799.00
	1999,Chevy,"Venture ""Extended Edition, Very Large""",,5000.00
	,,Venture "Extended Edition",,4900.00


```

If you want to write the same content in fixed width format, all you need is to create an instance of [FixedWidthWriter](./src/main/java/com/univocity/parsers/fixed/FixedWidthWriter.java) instead. The remainder of the code remains the same.

This will be the case for any other writers/parsers we might introduce in the future, and applies to all examples presented here.

#### Writing row by row, with comments ####


```java

	
	CsvWriterSettings settings = new CsvWriterSettings();
	// Sets the character sequence to write for the values that are null.
	settings.setNullValue("?");
	
	//Changes the comment character to -
	settings.getFormat().setComment('-');
	
	// Sets the character sequence to write for the values that are empty.
	settings.setEmptyValue("!");
	
	// writes empty lines as well.
	settings.setSkipEmptyLines(false);
	
	// Creates a writer with the above settings;
	CsvWriter writer = new CsvWriter(outputWriter, settings);
	
	// writes the file headers
	writer.writeHeaders("a", "b", "c", "d", "e");
	
	// Let's write the rows one by one (the first row will be skipped)
	for (int i = 1; i < rows.size(); i++) {
		// You can write comments above each row
		writer.commentRow("This is row " + i);
		// writes the row
		writer.writeRow(rows.get(i));
	}
	
	// we must close the writer. This also closes the java.io.Writer you used to create the CsvWriter instance
	// note no checked exceptions are thrown here. If anything bad happens you'll get an IllegalStateException wrapping the original error.
	writer.close();
	


```

The output of the above code should be:


```

	a,b,c,d,e
	-This is row 1
	1999,Chevy,Venture "Extended Edition",!,4900.00
	-This is row 2
	1996,Jeep,Grand Cherokee,"MUST SELL!
	...


```

#### Writing with column selection ####

You can write transparently to *some* fields of a CSV file, while keeping the output format consistent. Let's say you have a CSV
file with 5 columns but only have data for 3 of them, in a different order. All you have to do is configure the file headers
and select what fields you have values for. 


```java

	
	CsvWriterSettings settings = new CsvWriterSettings();
	
	// when writing, nulls are printed using the empty value (defaults to "").
	// Here we configure the writer to print ? to describe null values.
	settings.setNullValue("?");
	
	// if the value is not null, but is empty (e.g. ""), the writer will can be configured to
	// print some default representation for a non-null/empty value
	settings.setEmptyValue("!");
	
	// Encloses all records within quotes even when they are not required.
	settings.setQuoteAllFields(true);
	
	// Sets the file headers (used for selection, these values won't be written automatically)
	settings.setHeaders("Year", "Make", "Model", "Description", "Price");
	
	// Selects which fields from the input should be written. In this case, fields "make" and "model" will be empty
	// The field selection is not case sensitive
	settings.selectFields("description", "price", "year");
	
	// Creates a writer with the above settings;
	CsvWriter writer = new CsvWriter(outputWriter, settings);
	
	// Writes the headers specified in the settings
	writer.writeHeaders();
	
	// writes each row providing values for the selected fields (note the values and field selection order must match)
	writer.writeRow("ac, abs, moon", 3000.00, 1997);
	writer.writeRow("", 4900.00, 1999); // NOTE: empty string will be replaced by "!" as per configured emptyQuotedValue.
	writer.writeRow("MUST SELL!\nair, moon roof, loaded", 4799.00, 1996);
	
	writer.close();
	


```

The output of such setting will be:


```

	"Year","Make","Model","Description","Price"
	"1997","?","?","ac, abs, moon","3000.0"
	"1999","?","?","!","4900.0"
	"1996","?","?","MUST SELL!
	...


```

#### Writing with value conversions (using ObjectRowWriterProcessor) ####

All writers have a settings object that accepts an instance of [RowWriterProcessor](./src/main/java/com/univocity/parsers/common/processor/RowWriterProcessor.java). 
Use the writer methods prefixed with "processRecord" to execute the [RowWriterProcessor](./src/main/java/com/univocity/parsers/common/processor/RowWriterProcessor.java) against your input. 

In the following example, we use [ObjectRowWriterProcessor](./src/main/java/com/univocity/parsers/common/processor/ObjectRowWriterProcessor.java) to execute custom value conversions on each element of a row of objects.
This object executes a sequence of [Conversion](./src/main/java/com/univocity/parsers/conversions/Conversion.java) actions on the row elements before they are written.


```java

	
	FixedWidthFieldLengths lengths = new FixedWidthFieldLengths(15, 10, 35);
	FixedWidthWriterSettings settings = new FixedWidthWriterSettings(lengths);
	
	// Any null values will be written as ?
	settings.setNullValue("nil");
	settings.getFormat().setPadding('_');
	settings.setIgnoreLeadingWhitespaces(false);
	settings.setIgnoreTrailingWhitespaces(false);
	
	// Creates an ObjectRowWriterProcessor that handles annotated fields in the TestBean class.
	ObjectRowWriterProcessor processor = new ObjectRowWriterProcessor();
	settings.setRowWriterProcessor(processor);
	
	// Converts objects in the "date" field using the yyyy-MMM-dd format.
	processor.convertFields(Conversions.toDate(" yyyy MMM dd "), Conversions.trim()).add("date");
	
	// Trims Strings at position 2 of the input row.
	processor.convertIndexes(Conversions.trim(), Conversions.toUpperCase()).add(2);
	
	// Sets the file headers so the writer knows the correct order when writing values taken from a TestBean instance
	settings.setHeaders("date", "quantity", "comments");
	
	// Creates a writer with the above settings;
	FixedWidthWriter writer = new FixedWidthWriter(outputWriter, settings);
	
	// Writes the headers specified in the settings
	writer.writeHeaders();
	
	// writes a Fixed Width row with the values set in "bean". Notice that there's no annotated
	// attribute for the "date" column, so it will just be null (an then converted to ? a )
	writer.processRecord(new Date(0), null, "  a comment  ");
	writer.processRecord(null, 1000, "");
	
	writer.close();
	


```

The output will be:


```

	date___________quantity__comments___________________________
	1970 Jan 01____nil_______A COMMENT__________________________
	nil____________1000_________________________________________


```

#### Writing annotated java beans ####

If you have a java class with fields annotated with the annotations defined in package `com.univocity.parsers.annotations`, you can use a [BeanWriterProcessor](./src/main/java/com/univocity/parsers/common/processor/BeanWriterProcessor.java)
to map its attributes directly to the output.

A [RowWriterProcessor](./src/main/java/com/univocity/parsers/common/processor/RowWriterProcessor.java) is just an interface that "knows" how to map a given object to a sequence of values. By default, uniVocity-parsers provides the [BeanWriterProcessor](./src/main/java/com/univocity/parsers/common/processor/BeanWriterProcessor.java) to map annotated beans to rows.

The following example writes instances of [TestBean](./src/test/java/com/univocity/parsers/examples/TestBean.java): 


```java

	
	FixedWidthFieldLengths lengths = new FixedWidthFieldLengths(10, 10, 35, 10, 40);
	FixedWidthWriterSettings settings = new FixedWidthWriterSettings(lengths);
	
	// Any null values will be written as ?
	settings.setNullValue("?");
	
	// Creates a BeanWriterProcessor that handles annotated fields in the TestBean class.
	settings.setRowWriterProcessor(new BeanWriterProcessor<TestBean>(TestBean.class));
	
	// Sets the file headers so the writer knows the correct order when writing values taken from a TestBean instance
	settings.setHeaders("amount", "pending", "date", "quantity", "comments");
	
	// Creates a writer with the above settings;
	FixedWidthWriter writer = new FixedWidthWriter(outputWriter, settings);
	
	// Writes the headers specified in the settings
	writer.writeHeaders();
	
	// writes a fixed width row with empty values (as nothing was set in the TestBean instance).
	writer.processRecord(new TestBean());
	
	TestBean bean = new TestBean();
	bean.setAmount(new BigDecimal("500.33"));
	bean.setComments("Blah,blah");
	bean.setPending(false);
	bean.setQuantity(100);
	
	// writes a Fixed Width row with the values set in "bean". Notice that there's no annotated
	// attribute for the "date" column, so it will just be null (an then converted to ?, as we have settings.setNullValue("?");)
	writer.processRecord(bean);
	
	// you can still write rows passing in its values directly.
	writer.writeRow(BigDecimal.ONE, true, "1990-01-10", 3, null);
	
	writer.close();
	


```

The resulting output of the above code should be: 


```

	amount    pending   date                               quantity  comments
	?         ?         ?                                  ?         ?
	500.33    no        ?                                  100       blah,blah
	1         true      1990-01-10                         3         ?


```