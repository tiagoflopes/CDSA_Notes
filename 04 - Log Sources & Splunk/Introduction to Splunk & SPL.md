## What is Splunk?

Splunk is a highly scalable, versatile and robust data analytics software solution known for its ability to ingest, index, analyze and visualize massive amounts of machine data.
It's architecture consists of several layers that work together. It can be divided into the following:
- Forwarders - Responsible for data collection, they gather machine data from various sources and forward it to the indexers.
	- Universal Forwarder (UF) - Lightweight agent that collects data and sends it to the indexers without any preprocessing.
	- Heavy Forwarder (HF) - Serves the purpose of collecting data from remote sources, for intensive data aggregation assignments.
- Indexers - Receive data, organize it and store it in indexes. They also process search queries from users and return results.
- Search Heads - Coordinate jobs dispatching them to the indexers and merging the results. They also provide an interface for users to interact with Splunk.
- Deployment Server - Manages the configuration for forwarders, distributing apps and updates.
- Cluster Master - Coordinates the activities of indexers in a clustered environment, ensuring data replication and search affinity.
- License Master - Manages the licensing details of the Splunk platform.
Splunk's key components include:
- Splunk Web Interface - Graphical interface through which users can interact with Splunk, carrying out tasks like searching, creating alerts, dashboards and reports.
- Search Processing Language (SPL) - The query language for Splunk, that allows searching, filtering and manipulation of the indexed data.
- Apps and Add-ons - Provide specific functionalities within Splunk, and extend capabilities or integrate with other systems. Can be found on [Splunkbase](https://splunkbase.splunk.com/).
- Knowledge Objects - Include fields, tags, event types, lookups, macros, data models and alerts that enhance the data in Splunk, making it easier to search and analyze.

## Splunk as a SIEM Solution

Works as a SIEM solution, but it empowers organizations to enhance their detection capabilities by leveraging User Behavior Analytics.
Let's assume that *main* is an index containing Windows Security and Sysmon logs.

1. Basic Searching
The keyword "search" is implicit, so no need to write it.
`search index="main" "UNKNOWN"`
`index="main" "*UNKNOWN*"`
This returns events that contain the term anywhere in the data.

2. Fields and Comparison Operators
Splunk identifies certain data as fields (like source, sourcetype, host, EventCode, etc), and users can define additional fields.
`index="main" EventCode!=1`

3. The fields command
This specifies which fields should be included or excluded in the results.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | fields - User`
This restricts the scope to Sysmon event logs, and gets all Event ID 1 logs, excluding the User field from the results.

4. The table command
Presents search results in a tabular format.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | table _time, host, Image`

5. The rename command
Renames a field in the results.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | rename Image as Process`

6. The dedup command
Removes duplicate events.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | dedup Image`

7. The sort command
Sorts the search results.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | sort - _time`
By timestamp, in descending order.

8. The stats command
Performs statistic operations.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 | stats count by _time, Image`
Returns a table where each row represents a unique combination of a timestamp and a process.

9. The chart command
Created a data visualization based on statistical operations.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 | chart count by _time, Image`
Same as stats, but now each process has its own column.

10. The eval command
Creates or redefines fields.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | eval Process_Path=lower(Image)`
Creates a new field which contains the lowercase version of the Image field.

11. The rex command
Extracts new fields from existing ones using regular expressions.
`index="main" EventCode=4662 | rex max_match=0 "[^%](?<guid>{.*})" | table guid`
It looks for substrings that begin with *{* and end with *}*. *\[^%]* ensures that the match doesn't begin with a *%* character. The captured value is assigned to the named capture group *guid*. *max_match=0* prevents from only extracting the first occurrence.

12. The lookup command
Enriches the data with external sources.
Suppose we have a CSV file named "malware_lookup.csv".
```csv
filename, is_malware
notepad.exe, false
cmd.exe, false
powershell.exe, false
sharphound.exe, true
randomfile.exe, ture
```
It should be added as a new lookup table: Settings - Lookups - Lookup table files - New Lookup Table File.

`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | rex field=Image "(?P<filename>[^\\\]+)$" | eval filename=lower(filename) | lookup malware_lookup.csv filename OUTPUTNEW is_malware | table filename, is_malware`

- `| rex field=Image "(?P<filename>[^\\\]+)$"`: This command is using the regular expression (regex) to extract a part of the Image field. The Image field in Sysmon EventCode=1 logs typically contains the full file path of the process. This regex is saying: Capture everything after the last backslash (which should be the filename itself) and save it as filename.
- `| eval filename=lower(filename)`: This command is taking the filename that was just extracted and converting it to lowercase. The lower() function is used to ensure the search is case-insensitive.
- `| lookup malware_lookup.csv filename OUTPUTNEW is_malware`: This command is performing a lookup operation using the filename as a key. The lookup table (malware_lookup.csv) is expected to contain a list of filenames of known malicious executables. If a match is found in the lookup table, the new field is_malware is added to the event, which indicates whether or not the process is considered malicious based on the lookup table. *<-- filename in this part of the query is the first column title in the CSV*.

In summary, the query is extracting the filenames of newly created processes, converting them to lowercase, comparing them against a list of known malicious filenames, and presenting the findings in a table.

`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | eval filename=mvdedup(split(Image, "\\")) | eval filename=mvindex(filename, -1) | eval filename=lower(filename) | lookup malware_lookup.csv filename OUTPUTNEW is_malware | table filename, is_malware | dedup filename, is_malware`
This one removes duplicates based on the filename and is_malware fields.

13. The inputlookup command
Retrieves data from a lookup file without joining it to the resusts
`| inputlookup malware_lookup.csv`

14. Time range
Limit searches to specific time periods.
`index="main" earliest=-7d EventCode!=1`

15. The transaction command
Used to group events that share common characteristics into transactions, often used to track session or user activities that span across multiple events.
`index="main" sourcetype="WinEventLog:Sysmon" (EventCode=1 OR EventCode=3) | transaction Image startswith=eval(EventCode=1) endswith=eval(EventCode=3) maxspan=1m | table Image |  dedup Image`
- `| transaction Image startswith=eval(EventCode=1) endswith=eval(EventCode=3) maxspan=1m`: The transaction command is used here to group events based on the Image field, which represents the executable or script involved in the event. This grouping is subject to the conditions: the transaction starts with an event where `EventCode` is `1` and ends with an event where `EventCode` is `3`. 

16. Subsearches
Compute a set of results that are then used in the outer search.
`index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 NOT [ search index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | top limit=100 Image | fields Image ] | table _time, Image, CommandLine, User, ComputerName`


Find below some excellent resources:
- [https://docs.splunk.com/Documentation/SCS/current/SearchReference/Introduction](https://docs.splunk.com/Documentation/SCS/current/SearchReference/Introduction)
- [https://docs.splunk.com/Documentation/SplunkCloud/latest/SearchReference/](https://docs.splunk.com/Documentation/SplunkCloud/latest/SearchReference/)
- [https://docs.splunk.com/Documentation/SplunkCloud/latest/Search/](https://docs.splunk.com/Documentation/SplunkCloud/latest/Search/)

## How to Identify the Available Data

##### Leverage SPL
To identify the available source types, we can run the following, after selecting a time range.
`| eventcount summarize=false index=* | table index`

```shell-session
| metadata type=sourcetypes
```

This search uses the `metadata` command, which provides us with various statistics about specified indexed fields. Here, we're focusing on `sourcetypes`. The result is a list of all `sourcetypes` in our Splunk environment, along with additional metadata such as the first time a source type was seen (`firstTime`), the last time it was seen (`lastTime`), and the number of hosts (`totalCount`).

For a simpler view, we can use the following search.

```shell-session
| metadata type=sourcetypes index=* | table sourcetype
```

Here, the `metadata` command retrieves metadata about the data in our indexes. The `type=sourcetypes` argument tells Splunk to return metadata about sourcetypes. The `table` command is used to present the data in tabular form.

```shell-session
| metadata type=sources index=* | table source
```

This command returns a list of all data sources in the Splunk environment.

Once we know our source types, we can investigate the kind of data they contain. Let's say we are interested in a sourcetype named `WinEventLog:Security`, we can use the table command to present the raw data as follows.

```shell-session
sourcetype="WinEventLog:Security" | table _raw
```

The `table` command generates a table with the specified fields as columns. Here, `_raw` represents the raw event data. This command will return the raw data for the specified source type.

Splunk automatically extracts a set of default fields for every event it indexes, but it can also extract additional fields depending on the source type of the data. To see all fields available in a specific source type, we can use the `fields` command.

```shell-session
sourcetype="WinEventLog:Security" | table *
```

This command generates a table with all fields available in the `WinEventLog:Security` sourcetype. However, be cautious, as the use of `table *` can result in a very wide table if our events have a large number of fields. This may not be visually practical or effective for data analysis.

A better approach is to identify the fields you are interested in using the `fields` command as mentioned before, and then specifying those field names in the `table` command. Example:

```shell-session
sourcetype="WinEventLog:Security" | fields Account_Name, EventCode | table Account_Name, EventCode
```

If we want to see a list of field names only, without the data, we can use the `fieldsummary` command instead.

```shell-session
sourcetype="WinEventLog:Security" | fieldsummary
```

This search will return a table that includes every field found in the events returned by the search (across the sourcetype we've specified). The table includes several columns of information about each field:

- `field`: The name of the field.
- `count`: The number of events that contain the field.
- `distinct_count`: The number of distinct values in the field.
- `is_exact`: Whether the count is exact or estimated.
- `max`: The maximum value of the field.
- `mean`: The mean value of the field.
- `min`: The minimum value of the field.
- `numeric_count`: The number of numeric values in the field.
- `stdev`: The standard deviation of the field.
- `values`: Sample values of the field.

We may also see:

- `modes`: The most common values of the field.
- `numBuckets`: The number of buckets used to estimate the distinct count.

Please note that the values provided by the `fieldsummary` command are calculated based on the events returned by our search. So if we want to see all fields within a specific `sourcetype`, we need to make sure our time range is large enough to capture all possible fields.

```shell-session
index=* sourcetype=* | bucket _time span=1d | stats count by _time, index, sourcetype | sort - _time
```

Sometimes, we might want to know how events are distributed over time. This query retrieves all data (`index=* sourcetype=*`), then `bucket` command is used to group the events based on the `_time` field into 1-day buckets. The `stats` command then counts the number of events for each day (`_time`), `index`, and `sourcetype`. Lastly, the `sort` command sorts the result in descending order of `_time`.

```shell-session
index=* sourcetype=* | rare limit=10 index, sourcetype
```

The `rare` command can help us identify uncommon event types, which might be indicative of abnormal behavior. This query retrieves all data and finds the 10 rarest combinations of indexes and sourcetypes.

```shell-session
index="main" | rare limit=20 useother=f ParentImage
```

This command displays the 20 least common values of the `ParentImage` field.

```shell-session
index=* sourcetype=* | fieldsummary | where count < 100 | table field, count, distinct_count
```

A more complex query can provide a detailed summary of fields. This search shows a summary of all fields (`fieldsummary`), filters out fields that appear in less than 100 events (`where count < 100`), and then displays a table (`table`) showing the field name, total count, and distinct count.

```shell-session
index=* | sistats count by index, sourcetype, source, host
```

We can also use the `sistats` command to explore event diversity. This command counts the number of events per index, sourcetype, source, and host, which can provide us a clear picture of the diversity and distribution of our data.

```shell-session
index=* sourcetype=* | rare limit=10 field1, field2, field3
```

The rare command can also be used to find uncommon combinations of field values. Replace `field1`, `field2`, `field3` with the fields of interest. This command will display the 10 rarest combinations of these fields.

By combining the above SPL commands and techniques, we can explore and understand the types of data source, the data they contain, and the fields within them. This understanding is the foundation upon which we build effective searches, reports, alerts, and dashboards in Splunk.

Lastly, remember to follow your organization's data governance policies when exploring data and source types to ensure you're compliant with all necessary privacy and security guidelines.

You can apply each of the searches mentioned above successfully against the data within the Splunk instance of the target that you can spawn below. We highly recommend running and even modifying these searches to familiarize yourself with SPL (Search Processing Language) and its capabilities.

##### Leverage the graphical interface

When using the `Search & Reporting` application's user interface, identifying the available data source types, the data they contain, and the fields within them becomes a task that involves interacting with various sections of the UI. Let's examine how we can effectively use the Splunk Web interface to identify these elements.

- `Data Sources`: The first thing we want to identify is our data sources. We can do this by navigating to the `Settings` menu, which is usually located on the top right corner of our Splunk instance. In the dropdown, we'll find `Data inputs`. By clicking on `Data inputs`, we'll see a list of various data input methods, including files & directories, HTTP event collector, forwarders, scripts, and many more. These represent the various sources through which data can be brought into Splunk. Clicking into each of these will give us an overview of the data sources being utilized.
    
- `Data (Events)`: Now, let's move on to identifying the data itself, in other words, the events. For this, we'll want to navigate to the `Search & Reporting` app. By exploring the events in the `Fast` mode, we can quickly scan through our data. The `Verbose` mode, on the other hand, lets us dive deep into each event's details, including its raw event data and all the fields that Splunk has extracted from it. In the search bar, we could simply put `*` and hit search, which will bring up all the data that we have indexed. However, this is usually a massive amount of data, and it might not be the most efficient way to go about it. A better approach might be to leverage the time picker and select a smaller time range (let's be careful while doing so though to not miss any important/useful historic logs).
    
- `Fields`: Lastly, to identify the fields in our data, let's look at an event in detail. We can click on any event in our search results to expand it. ![[splunk_fields.webp]] We can also see on the left hand side of the "Search & Reporting" application two categories of fields: `Selected Fields` and `Interesting Fields`. `Selected Fields` are fields that are always shown in the events (like `host`, `source`, and `sourcetype`), while `Interesting Fields` are those that appear in at least 20% of the events. By clicking `All fields`, we can see all the fields present in our events.
    
- `Data Models`: Data Models provide an organized, hierarchical view of our data, simplifying complex datasets into understandable structures. They're designed to make it easier to create meaningful reports, visualizations, and dashboards without needing a deep understanding of the underlying data sources or the need to write complex SPL queries. Here is how we can use the Data Models feature to identify and understand our data:
    
    - `Accessing Data Models`: To access Data Models, we click on the `Settings` tab on the top right corner of the Splunk Web interface. Then we select `Data Models` under the `Knowledge` section. This will lead us to the Data Models management page. `<-- If it appears empty, please execute a search and navigate to the Data Models page again.`
    - `Understanding Existing Data Models`: On the Data Models management page, we can see a list of available Data Models. These might include models created by ourselves, our team, or models provided by Splunk Apps. Each Data Model is associated with a specific app context and is built to describe the structured data related to the app's purpose.
    - `Exploring Data Models`: By clicking on the name of a Data Model, we are taken to the `Data Model Editor`. This is where the true power of Data Models lies. Here, we can view the hierarchical structure of the data model, which is divided into `objects`. Each object represents a specific part of our data and contains `fields` that are relevant to that object. For example, if we have a Data Model that describes web traffic, we might see objects like `Web Transactions`, `Web Errors`, etc. Within these objects, we'll find fields like `status`, `url`, `user`, etc.
 
- `Pivots`: Pivots are an extremely powerful feature in Splunk that allows us to create complex reports and visualizations without writing SPL queries. They provide an interactive, drag-and-drop interface for defining and refining our data reporting criteria. As such, they're also a fantastic tool for identifying and exploring the available data and fields within our Splunk environment. To start with Pivots to identify available data and fields, we can use the `Pivot` button that appears when we're browsing a particular data model in the `Data Models` page.
