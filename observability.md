30 Claude Code Prompts for Complete Observability
Data Flow & Transformation Observability
1.	"Generate a function that tracks data transformations through my pipeline. Log the record count, data types, and schema changes at each step. Output a JSON report showing data lineage with before/after row counts and any type mismatches."
2.	"Create an observability wrapper for my data processing function that captures: input data summary (count, memory), processing time in milliseconds, output statistics, and any rows that were dropped or transformed. Generate an HTML dashboard showing these metrics."
3.	"Write code that monitors data flowing between my source database and data warehouse. Track: record counts at source, records in transit, records loaded to target, and percentage completion. Generate a real-time CSV export showing data freshness by table."
4.	"Build a validation observer that checks data completeness, consistency, and accuracy at each pipeline stage. Log field-level statistics (nulls, duplicates, value ranges) and generate a CSV report flagging any anomalies with severity levels."
API Observability & Response Validation
5.	"Create an API response observer that logs: request latency (p50, p95, p99), status code distribution, response payload size, and error messages. Generate an HTML report with graphs showing response time trends and error rate percentiles."
6.	"Generate monitoring code that captures all HTTP requests/responses including headers, body content, and timing. Create a searchable JSON log and an HTML report showing API endpoint performance, failures, and response data samples."
7.	"Write code that validates API response schemas, tracks what fields are present/missing, checks data types, and monitors value distributions. Output a CSV showing response structure consistency and any deviations from expected schema."
8.	"Build an API call tracer that logs every request parameter, response payload, HTTP headers, and timing information. Generate an interactive HTML report showing request chains, failed calls highlighted in red, and response time heatmaps."
Test Execution & Code Path Observability
9.	"Create test observability that logs: which code paths were executed, function call sequences, variable values at key points, and execution time per function. Generate an HTML report showing code coverage and execution flow diagrams."
10.	"Generate a test runner observer that captures: test case name, pass/fail status, execution time, assertion details, and any exceptions thrown. Create both a CSV summary and an HTML report with color-coded results and failure stack traces."
11.	"Write code that traces every decision point in my code (if statements, loops, branches). Log how many times each branch executes, what values triggered each path, and generate a visual flowchart showing execution distribution."
12.	"Build an observer that logs every function call with: function name, input parameters, return value, execution time, and any exceptions. Generate a JSON call tree showing function hierarchy and a CSV with performance metrics per function."
Database Query Observability
13.	"Create a database query observer that logs: SQL query text, execution time in milliseconds, rows affected/returned, query parameters, and any database errors. Generate an HTML report showing slow queries, most frequently executed queries, and a timeline of query execution."
14.	"Write code that monitors: database connection status, active connections, query queue size, and response times. Log transaction start/commit/rollback events. Generate a CSV showing database health metrics and an HTML dashboard with connection pool utilization."
15.	"Generate observability for database write operations: log affected row counts before/after, changed values, timestamps, and any constraint violations. Create a CSV audit trail and an HTML report showing data modification patterns."
Error & Exception Observability
16.	"Create an error observer that captures: error type, message, stack trace, local variables at point of failure, and recovery attempt status. Generate an HTML report grouping errors by type with frequency counts and affected code sections highlighted."
17.	"Write code that logs every try-catch block: what exception was caught, what recovery action was taken, whether it succeeded, and how long recovery took. Generate a JSON report showing error recovery patterns and failure rates."
18.	"Build an observer that tracks error propagation through your call stack. Log at which level each error was caught/handled, what context information was available, and what action was taken. Create a flowchart showing error handling paths."
Performance & Resource Observability
19.	"Create a performance profiler that logs: CPU time, memory usage (before/after), garbage collection events, and I/O operations per code block. Generate an HTML flamegraph and CSV showing hotspots where most time/memory is consumed."
20.	"Write code that monitors: function execution time, memory allocated per function, and identifies performance bottlenecks. Generate an HTML report ranking functions by execution time and an interactive graph showing memory usage over time."
21.	"Generate an observer that tracks: loop iterations, conditional branch taken counts, cache hit/miss rates (if applicable), and generates a CSV showing which code sections consume most resources relative to their size."
Third-Party Service Observability
22.	"Create an observer for external API calls that logs: endpoint URL, request/response payloads, HTTP status codes, latency, timeout events, and retry attempts. Generate an HTML dashboard showing external service health and response time trends."
23.	"Write code that monitors: file I/O operations (open, read, write, close), file sizes, access times, and error conditions. Generate a CSV showing file operation patterns and an HTML report highlighting slow file operations."
24.	"Build an observer that logs: network calls made, response times, data transferred (bytes), error codes, and retry logic. Generate a JSON trace showing network call sequences and an HTML visualization of network dependencies."
Configuration & State Observability
25.	"Create an observer that logs: initial configuration values, where configuration comes from (env vars, files, defaults), when configuration is changed, and what the old/new values are. Generate a JSON configuration audit trail and HTML report."
26.	"Write code that tracks application state changes: what state values changed, when they changed, what triggered the change, and what the previous state was. Generate an HTML timeline showing state transitions with before/after snapshots."
27.	"Generate an observer that logs environment variables, system resources available, runtime version information, and detects any configuration drift. Create an HTML report comparing expected vs. actual environment setup."
Integration & Data Sync Observability
28.	"Create an observer for data sync operations that logs: records synced, sync status (pending/synced/failed), timing information, and data consistency checks between systems. Generate an HTML dashboard showing sync health and a CSV identifying failed syncs."
29.	"Write code that monitors: cache operations (hits, misses, evictions), cache size over time, and data consistency between cache and source. Generate a JSON report showing cache effectiveness and an HTML graph of cache performance metrics."
30.	"Build an observer that logs every message/event processed: timestamp, source, content, processing result, and any transformations applied. Generate an HTML report showing message flow patterns, processing latency distribution, and any processing failures."
________________________________________
How to Use These Prompts with Claude Code
Format your request:
"Generate [component] with observability. Track: [metrics to track]
Output formats: [CSV/JSON/HTML/Dashboard]. 
Create reports showing: [specific insights]"
Pro tips:
•	Combine prompts: "Use prompt #5 for API calls AND prompt #13 for database queries in the same function"
•	Request specific outputs: Ask for both structured data (CSV/JSON) AND visual reports (HTML)
•	Add real-time elements: "Include a streaming CSV that updates as code runs" or "Generate a live HTML dashboard"
•	Chain observability: "After running prompt #9, generate test observability (prompt #10), then create an HTML report combining both"
Common metric combinations to request:
•	Latency (p50, p95, p99) + Error rates + Throughput = API health
•	Record counts + Data types + Schema validation = Data quality
•	Function execution time + Memory usage + Error count = Performance health
•	Request/response payloads + Status codes + Retry logic = Integration health
The key insight: AI doesn't just write your features—prompt it to write the complete instrumentation that proves those features work. 
