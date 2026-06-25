---
name: awesome-enterprise-shortcodes
description: Programming guidelines and service reference for developing in the AwesomeEnterprise low-code framework using shortcodes.
---

## Core Concepts & Processor Architecture

AwesomeEnterprise is a custom low-code templating platform. It processes code containing comments, shortcodes, and plain text, and outputs a single text stream.

### The Execution Lifecycle
When a block of code is parsed by the Awesome Processor, it executes in three steps:
1. **Comment Removal**: All comments (`//* ... *//`) are stripped out.
2. **First-Level Shortcode Execution**: Every first-level shortcode is evaluated, resolved, and replaced with its text output. If a shortcode returns an array or object directly, the processor outputs its PHP `print_r` textual representation. Thus, you must explicitly format arrays/objects or use Outputters.
3. **Stream Output**: The parsed text with evaluated shortcodes is returned.

### Shortcode Execution Order
Every shortcode executes in four distinct processor stages:
1. **Conditions Processor (`c.*`, `and.*`, `or.*`)**: Evaluates conditional attributes. If they resolve to `false`, the shortcode is skipped entirely.
2. **Service Processor**: Runs the core service using the provided service attributes and inner content. Returns a result value of any data type.
3. **Modifiers Processor (`m.*`, `m2.*`)**: Sequentially modifies the value returned by the service processor.
4. **Outputters Processor (`o.*`, `o2.*`)**: Directs the final modified value (e.g., sets an env variable, dumps output, or deletes the value to return an empty string). If no outputter is present, the modified value is printed directly to the output stream.

---

## Syntax & Datatypes

### Comments
Comments use `//*` to start and `*//` to end. They can be single-line or multi-line:
```
//* This is a single-line comment *//

//*
This is a
multiline comment
*//
```

### Shortcode Structure
Shortcodes can be self-closing:
```
[str.create "Vikas Kumar" /]
```
Or they can wrap inner content (enclosed between opening and closing tags):
```
[if.equal lhs="6" rhs="6"]
  we are equal
[/if.equal]
```

### Datatype Castings
By default, all shortcode attribute values are treated as strings. To enforce specific datatypes, use typecast prefixes in service attributes:
*   `int:<value>`: Casts value to an integer (e.g. `int:42`).
*   `num:<value>` or `float:<value>`: Casts value to a float (e.g. `num:22.99`).
*   `str:<value>`: Casts value to a string (e.g. `str:hello`).
*   `bool:<value>`: Converts `"false"` to `false`, else casts to `bool` (e.g. `bool:false`).
*   `comma:<val1,val2>`: Converts a comma-separated list into a string array (e.g. `comma:a,b,c`).
*   `get:<path>`: Retrieves the value at the given environment path and preserves its original datatype.
*   `x:<service_call>`: Executes a service and returns the output with its datatype preserved (e.g., `x:request2.get name`).

### Variables & Service References in Attributes
*   **Curly Braces `{path.key}`**: Evaluates the environment variable path and inserts its value. Nested curly braces are not allowed (e.g., `{person.{name}}` is invalid). Spaces are not allowed inside curly braces.
*   **Double Curly Braces `{{service.name}}`**: Executes the specified service call and returns the result value. It is recommended to wrap double curly braces in double quotes (e.g., `"{{request2.get name}}"`).
*   **Important**: Curly braces `{}` and `{{}}` are **only parsed within attributes**, not inside the content of shortcodes. To print variables or execute code inside content, write full shortcodes (e.g. `[env.get path /]`).

---

## Variable Scoping Scenarios

To prevent cluttering the global scope, AwesomeEnterprise supports three scopes:

### 1. Module Scope (`module.*`)
Active when a module is running. It stores module state and parameters inside a temporary `module` state variable in the environment.
*   Parameters passed to a module (`[collection.module param1=val /]`) are automatically attached to the `module` state variable.
*   Services like `[module.get]`, `[module.set]`, and `[module.set_array]` read and write to this local scope.
*   Use `[module.return <env_path> /]` to exit the module early and return the value at `<env_path>`.

### 2. Template Scope (`template.*`)
Active when a template runs within a module. It stores local parameters passed directly to the template.
*   Use `[template.get]`, `[template.set]`, and `[template.set_array]` to manage template variables.
*   Use `[template.return <env_path> /]` to exit early and return the value at `<env_path>`.

### 3. Block Scope (`do.*`)
Active inside a `[do.@local_var]` block. The local variable `@local_var` is destroyed immediately when the closing `[/do.@local_var]` is reached.
```
[do.@temp]
    [env.set @temp.x="Hello World" /]
    [env.get @temp.x /]
[/do.@temp]
//* At this point, @temp.x no longer exists *//
```

---

## Core API Library Reference

### Env Library
Used to interact with the environment state.
*   `[env.set key=val /]`: Sets one or more variables in the environment.
*   `[env.get path /]`: Returns the value at `path`.
*   `[env.set_raw path]content[/env.set_raw]`: Saves raw, unparsed content at `path` to be evaluated later.
*   `[env.set_array path]...[/env.set_array]`: Builds nested multidimensional associative or indexed arrays.
*   `[env.dump path /]`: Calls PHP `var_dump` on the value at `path`.
*   `[env.echo path /]`: Prints the value at `path` using PHP `echo`.

### Loop Library
Allows range-based or array-based iterations.
*   **Array Loop**: `[loop.@iterator array_path] ... [/loop.@iterator]`
    Exposes these variables inside the loop:
    *   `[env.@iterator.item]`: The current item value. If the array contains objects, nested properties of the current item **must** be accessed by prefixing them with `.item` (e.g. `[env.@iterator.item.property /]` or inside attributes as `{@iterator.item.property}`).
        *   **CRITICAL RULE**: Never access properties directly on the iterator itself (e.g., `{@iterator.property}` or `[env.@iterator.property /]` is **invalid** and will fail). You must go through `.item` (e.g., `{@iterator.item.property}`).
    *   `[env.@iterator.key]`: The current key.
    *   `[env.@iterator.index]`: 1-based index.
    *   `[env.@iterator.counter]`: 0-based index.
    *   `[env.@iterator.first]`, `[env.@iterator.last]`, `[env.@iterator.between]`: Boolean flags.
    *   `[env.@iterator.odd]`, `[env.@iterator.even]`: Boolean flags.
*   **Range Loop**: `[loop.@iterator start="1" stop="5" step="1"] ... [/loop.@iterator]`
    Exposes the same metadata fields. If `start` < `stop`, increments by `step`; if `stop` < `start`, decrements.

### If Library
Handles conditional checks and branching.
*   `[if.equal lhs=A rhs=B] ... [/if.equal]`: Checks if `lhs` equals `rhs`.
*   `[if.not_equal lhs=A rhs=B] ... [/if.not_equal]`: Checks if `lhs` does not equal `rhs`.
*   `[if.else] ... [/if.else]`: Executes if the preceding `if` condition fails.
*   `[if.arr path]`: True if the variable at `path` is an array.
*   `[if.contains needle=val haystack={array_val}]`: True if `needle` is in the `haystack` array.
*   `[if.not_contains needle=val haystack={array_val}]`: True if `needle` is not in the `haystack` array.
*   `[if.empty path]`, `[if.not_empty path]`: Checks if string or array is empty/not empty.
*   `[if.int path]`, `[if.num path]`, `[if.not_int path]`, `[if.not_num path]`: Type checking.

### Math Library
Performs mathematical calculations.
*   `[math.solve "expression" /]`: Evaluates math expressions. Power operation uses `**` (e.g., `2**8` for $2^8$). Evaluates literals, not env paths directly.
*   `[math.minus_one "val" /]`: Decrements value by 1.
*   `[math.plus_one "val" /]`: Increments value by 1.

### Str, Int, Bool, Num, and Date Libraries
*   `[str.get path default=val /]`, `[str.create "val" /]`: Gets or creates string values.
*   `[int.get path default=0 /]`, `[int.create 42 /]`: Gets or creates integers.
*   `[bool.get path default=false /]`, `[bool.create "true" /]`: Gets or creates booleans.
*   `[num.get path default=0.0 /]`, `[num.create "2.00" /]`: Gets or creates floats.
*   `[date.get path default=val /]`, `[date.create "2026-05-20" /]`: Gets or creates DateTime objects.
*   `[date.modify date=date_val frequency="+1 day" /]`: Modifies date.
*   `[date.diff date_from=A date_to=B type="days" /]`: Calculates date difference (`mins`, `hours`, `days`, `years`, or `english`).

### Arr Library
*   `[arr.set path key1=val1 /]`: Sets array keys.
*   `[arr.create] ... [/arr.create]`: Builds array from content.
*   `[arr.empty o.set=path /]`: Returns an empty array.
*   `[arr.search_deep main=array_path search=value field=field_name /]`: Recursively searches deep arrays/objects.

### Request2 Library
Safely reads input from PHP `$_REQUEST`, filtering for XSS.
*   `[request2.get "key" /]`: Cleans and returns the value of "key" from request.
    *   If `key` is `"request_body"`, returns raw request body (`php://input`).
    *   If `key` is `"post_json"`, returns sanitised JSON string of `$_POST`.
*   `[request2.<key> /]`: Shorthand syntax for `[request2.get "<key>" /]`.

### Curl Library
Used for executing HTTP requests. Returns an array with keys `info`, `headers`, `cookies`, and `result`.
*   `[curl.api.get url=... data=... header=... proxy=... /]`
*   `[curl.api.post url=... data=... header=... proxy=... /]`
*   `[curl.page.get url=... data=... header=... proxy=... /]`
*   `[curl.api.patch url=... data=... header=... proxy=... /]`

### Mysqli Library (Single Query)
*   `[mysqli.cud]query[/mysqli.cud]`: Fires an INSERT, UPDATE, or DELETE query.
*   `[mysqli.fetch.rows]query[/mysqli.fetch.rows]`: Returns multiple rows in a `rows` array.
*   `[mysqli.fetch.row]query[/mysqli.fetch.row]`: Returns exactly 0 or 1 row.
*   `[mysqli.fetch.exactly_one_row]query[/mysqli.fetch.exactly_one_row]`: Returns exactly 1 row. Throws an error otherwise.
*   `[mysqli.fetch.col]query[/mysqli.fetch.col]`: Returns a single column as an array.
*   `[mysqli.fetch.scalar]query[/mysqli.fetch.scalar]`: Returns exactly 1 row and 1 column value.
*   `[mysqli.fetch.count]query[/mysqli.fetch.count]`: Returns a numeric count.

### Mysqli.multi Library (Multi Queries & Transactions)
Used to fire multiple queries in a single call.
*   `[mysqli.multi.cud]...[/mysqli.multi.cud]`: Executes multi-statement CUD queries.
*   `[mysqli.multi.fetch.rows]...[/mysqli.multi.fetch.rows]`: Executes multi-statements and returns the last resultset.
*   `[mysqli.transaction.commit]...[/mysqli.transaction.commit]`: Runs queries in an isolated SQL transaction, automatically committing them.
*   `[mysqli.transaction.rollback]...[/mysqli.transaction.rollback]`: Runs queries and rolls back at the end.

---

## Security & Database Escaping

To prevent SQL Injection and XSS attacks, **never concatenate raw values directly into SQL query strings**. Instead:
1. Escape all variables using the `esc` library.
2. Set them as MySQL variables at the beginning of a multi-query.
3. Reference the MySQL variables inside the SQL statements.

### Esc Library Reference (MySQL queries only)
All `esc` services receive an **environment variable path** (not a literal value) as their main attribute.
*   `[esc.int path /]`: Casts path value to safe integer.
*   `[esc.num path /]`: Casts path value to safe float.
*   `[esc.table path /]`: Escapes table name string.
*   `[esc.str path /]`: Escapes string value and wraps it in single quotes.
*   `[esc.id path /]`: Sanitises typical IDs (e.g. `B89`), wrapping the result in single quotes.
*   `[esc.date path /]`: Converts date to YYYYMMDD and returns `DATE('escaped_date')`.
*   `[esc.like path /]`: Escapes LIKE query wildcards (`%` and `_`). Does not add single quotes.
*   `[esc.unsafe path /]`: Converts numeric array to a comma-separated string (e.g., `54,45,67`).
*   `[esc.unsafe.quotes path /]`: Converts string array to comma-separated single-quoted escaped strings (e.g. `'hello','world'`).
*   `[esc.safe path /]`, `[esc.safe.quotes path /]`: Acts as a wrapper indicating values are already escaped.

### Safe SQL Query Pattern
```sql
[mysqli.multi.fetch.rows o.set="template.results"]
    SET @category = [esc.str request.category /];
    SET @status = 'publish';

    SELECT ID, post_title 
    FROM wp_posts 
    WHERE post_status = @status AND post_name = @category;
[/mysqli.multi.fetch.rows]
```

*Note: MySQL does not allow MySQL variables inside the `LIMIT` clause. Limit values must be hardcoded in the query string itself.*

---

## Modifiers, Conditions & Outputters (Cheat Sheet)

### Modifiers (`m.*`)
*   `m.run=yes`: Executes unparsed code stored in a string variable.
*   `m.lower=yes`, `m.upper=yes`, `m.capitalize=yes`, `m.sentence=yes`: Case conversions.
*   `m.trim=yes`, `m.ltrim=yes`, `m.rtrim=yes`: Trims whitespaces.
*   `m.left=i:N`, `m.right=i:N`: Truncates characters from left or right.
*   `m.substr=yes m.start=i:X m.chars=i:Y`: Extracts substring.
*   `m.length=yes`: String length or array count.
*   `m.json_encode=yes`, `m.json_decode=yes`: JSON conversions.
*   `m.explode_on=s:comma/space/dot/<delim>`: Explodes string into array.
*   `m.implode_on=s:comma/space/dot/quote_comma/<delim>`: Implodes array into string.
*   `m.number_format=yes m.decimals=n:X`: Formats a number with decimals.
*   `m.money_format='en_IN' m.currency='INR'`: Currency formatting.
*   `m.date_format='M d, Y'`: Formats date strings.
*   `m.hhmm_format=yes`: Formats seconds into HH:MM:SS.
*   `m.words=n:X` / `m.words=yes`: Limits words in string.
*   `m.url_encode=yes`, `m.url_decode=yes`: URL encoding/decoding.
*   `m.arr_item=s:first/last/key` or `m.arr_item=n:index`: Extracts array item.
*   `m.unset_item=first/last/key`: Removes array item.
*   `m.shuffle=yes`: Shuffles array items.
*   `m.solve=yes`: Evaluates string as mathematical expression.
*   `m.to_str=yes`, `m.to_int=yes`, `m.to_num=yes`, `m.to_bool=yes`: Type conversion.
*   `m.append='str'`, `m.prepend='str'`: Appends/prepends strings.
*   `m.empty_array=yes`: Clears array.
*   `m.sort='asort/arsort/ksort/rsort'`: Sorts array.

### Conditions (`c.*`)
*   `c.ignore=yes`: Always skips shortcode.
*   `c.odd=i:N`, `c.even=i:N`: Checks parity of number.
*   `c.cond=A c.eq=B`: True if A === B.
*   `c.cond=A c.neq=B`: True if A !== B.
*   `c.cond=A c.gt=B`, `c.cond=A c.lt=B`, `c.cond=A c.gte=B`, `c.cond=A c.lte=B`: Comparison filters.
*   `c.true=b:val`, `c.false=b:val`: Boolean validations.
*   `c.yes=val`, `c.no=val`: Checks if value is the string `"yes"` or `"no"`.
*   `c.arr=path`, `c.not_arr=path`: Checks array status.
*   `c.empty=path`, `c.not_empty=path`: Empty validations.
*   `c.null=path`, `c.not_null=path`: Null validations.
*   `c.contains=needle c.haystack={arr}`: Checks array content.
*   `c.exists=key`, `c.not_exists=key`: Checks environment paths.
*   `c.logged_in=yes`, `c.not_logged_in=yes`: User session state.
*   `c.user_can=role`, `c.user_cannot=role`: User role permissions.
*   `c.device='mobile,tablet'`: Matches request user device.

### Outputters (`o.*`)
*   `o.die=yes`: Outputs value and calls PHP `die()`.
*   `o.echo=yes`: Echoes value to standard output stream (retains value for subsequent processors).
*   `o.dump=yes`: Prints HTML representation of the value (retains value).
*   `o.console=yes`: Logs output in browser console (retains value).
*   `o.log=yes`: Logs output in error logs (retains value).
*   `o.destroy=yes`: Destroys the value and returns empty string.
*   `o.set='path'`: Saves the value at the env path and returns empty string.
*   `o.merge_with='path'`: Merges array value with target array at env path.

---

## Common Pitfalls & Rules

1.  **Single Quotes in Unnamed Attributes**: Do not use single quotes to wrap values inside unnamed/main attributes. Use double quotes or no quotes instead.
    *   *Invalid*: `[str.create 'hello' /]`
    *   *Valid*: `[str.create "hello" /]` or `[str.create hello /]`
2.  **Shortcodes Inside Attributes**: Do not nest square brackets inside shortcode attributes. Use curly braces to fetch variables.
    *   *Invalid*: `[env.set x=[env.get y] /]`
    *   *Valid*: `[env.set x={y} /]`
3.  **Untrapped Output Streams**: Shortcodes that return data (e.g., `mysqli.fetch.rows` or `curl.api.get`) will print their text/PHP debug outputs directly to the page unless trapped with `o.set` or deleted with `o.destroy`.
    *   *Invalid*: `[mysqli.fetch.rows]SELECT * FROM wp_posts[/mysqli.fetch.rows]` (dumps array output to page)
    *   *Valid*: `[mysqli.fetch.rows o.set="template.posts"]SELECT * FROM wp_posts[/mysqli.fetch.rows]`
4.  **Shortcode-in-Shortcode Syntax Error**: Nested square brackets in service attributes will fail to parse.
    *   *Invalid*: `[env.set x=[esc.str y] /]`
    *   *Valid*: `[esc.str y o.set="temp.esc_y" /][env.set x={temp.esc_y} /]`
5.  **Nested Curly Braces**: Variable syntax `{}` cannot contain other curly braces.
    *   *Invalid*: `{person.{name_field}}`
    *   *Valid*: Find the nested value outside first, set it to a flat variable, then reference that flat variable.
6.  **Curly Braces in Content**: `{}` and `{{}}` are not evaluated within inner shortcode content; they will print literally. Use appropriate shortcodes (e.g., `[env.get ...]`) in content areas.
7.  **Limit Variables in MySQL**: Do not pass MySQL variables (e.g., `@limit`) into the `LIMIT` clause of SQL queries. Use hardcoded limit numbers in the query template.
8.  **Strict Service Guidelines**: Never use functions or services not explicitly documented in the framework guidelines.
9.  **Loop Variable Access (Strict Scoping)**: When iterating with `[loop.@iterator array_path]`, the properties of the current item are scoped under the `.item` namespace.
    *   *Invalid (Direct property access)*: `{@iterator.name}` or `[env.@iterator.name /]`
    *   *Valid (Correct scoped access)*: `{@iterator.item.name}` or `[env.@iterator.item.name /]`

