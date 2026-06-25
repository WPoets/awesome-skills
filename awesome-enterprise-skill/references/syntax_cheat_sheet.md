# AwesomeEnterprise Syntax Cheat Sheet

This reference guide provides quick syntax templates, condition maps, modifier formats, and outputters for developing in the AwesomeEnterprise low-code framework.

## 1. Variable Typecasting Prefix Cheat Sheet
Apply these prefixes within attributes to force specific types:

| Prefix | Typecast Target | Example |
| :--- | :--- | :--- |
| `int:` | Integer | `[env.set age=int:30 /]` |
| `num:` / `float:` | Float | `[env.set price=num:19.99 /]` |
| `str:` | String | `[env.set name=str:hello /]` |
| `bool:` | Boolean | `[env.set is_active=bool:false /]` |
| `comma:` | Exploded String Array | `[env.set roles=comma:admin,user /]` |
| `get:` | Env Value (Preserves Type) | `[env.set current_val=get:some.path /]` |
| `x:` | Dynamic Service Evaluation | `[env.set name="x:request2.get username" /]` |

---

## 2. Common Shortcode Syntax Templates

### Setting & Getting Variables
```xml
[env.set x=1 y="hello dolly" is_active=bool:true /]
[env.get x /]
```

### Isolated Local Scope Code Blocks
```xml
[do.@temp]
    [env.set @temp.user_id="U123" /]
    [env.get @temp.user_id /]
[/do.@temp]
```

### Conditional Branches
```xml
[if.equal lhs="{template.role}" rhs="admin"]
    <p>Welcome Admin!</p>
[/if.equal]
[if.else]
    <p>Access Denied.</p>
[/if.else]
```

### Looping Over an Array
```xml
[loop.@i env.items]
    //* CRITICAL: Properties must be accessed via .item (e.g. env.@i.item.name), NOT env.@i.name *//
    <p>Index: [env.@i.index /] | Item: [env.@i.item.name /]</p>
[/loop.@i]
```

### Multi-Query Safe SQL Template
```xml
[mysqli.multi.fetch.rows o.set="template.user_posts"]
    SET @author_id = [esc.int request.author /];
    SET @status = 'publish';
    
    SELECT ID, post_title, post_date 
    FROM wp_posts 
    WHERE post_author = @author_id 
      AND post_status = @status;
[/mysqli.multi.fetch.rows]
```

---

## 3. Conditions Reference (`c.*`)
Conditions evaluate dynamically. If the check fails, the shortcode is skipped.

| Attribute Check | Description |
| :--- | :--- |
| `c.ignore=yes` | Skip shortcode execution completely |
| `c.odd=i:X` / `c.even=i:X` | Check integer parity |
| `c.cond=A c.eq=B` / `c.neq=B` | Check type and value equality/inequality |
| `c.cond=A c.gt=B` / `c.lt=B` | Greater than / Less than comparison |
| `c.cond=A c.gte=B` / `c.lte=B` | Greater or equal / Less or equal comparison |
| `c.true=b:X` / `c.false=b:X` | Boolean check |
| `c.yes=X` / `c.no=X` | String matches `"yes"` or `"no"` |
| `c.arr=path` / `c.not_arr=path` | Check if environment variable at path is/is not an array |
| `c.empty=path` / `c.not_empty=path` | Check if path value is empty/not empty |
| `c.null=path` / `c.not_null=path` | Check if path value is null/not null |
| `c.contains=needle c.haystack={arr}`| Check if array in haystack contains needle |
| `c.logged_in=yes` / `c.not_logged_in=yes`| Checks system login state |
| `c.user_can=role` / `c.user_cannot=role`| Checks user credentials |

---

## 4. Modifiers Reference (`m.*`)
Modifiers format or manipulate the returned shortcode values.

| Modifier | Value Argument | Description |
| :--- | :--- | :--- |
| `m.lower=yes` / `m.upper=yes`| None | Convert text to lowercase/uppercase |
| `m.capitalize=yes` / `m.sentence=yes` | None | Capitalise string / convert to sentence case |
| `m.trim=yes` / `m.ltrim=yes` / `m.rtrim=yes` | None | Trim whitespace |
| `m.left=i:N` / `m.right=i:N` | `i:Length` | Truncate characters from left or right |
| `m.substr=yes` | `m.start=i:X m.chars=i:Y` | Cut substring |
| `m.length=yes` | None | Get string length or array count |
| `m.json_encode=yes` / `m.json_decode=yes` | None | JSON conversion |
| `m.explode_on=s:delim` | `s:comma` / `s:space` etc. | Explode string to array |
| `m.implode_on=s:delim` | `s:comma` / `s:quote_comma` | Implode array to string |
| `m.number_format=yes` | `m.decimals=n:X` | Format floats with decimal count |
| `m.money_format='en_IN'` | Optional: `m.currency='INR'` | Currency formatting |
| `m.date_format='M d, Y'` | Format string | Format dates |
| `m.hhmm_format=yes` | None | Format seconds into HH:MM:SS |
| `m.words=n:X` / `m.words=yes` | `n:Count` | Limit word count |
| `m.url_encode=yes` / `m.url_decode=yes` | None | URL encoding/decoding |
| `m.arr_item=s:val` / `m.arr_item=n:val` | `s:first` / `s:last` / `n:index` | Extract specific array item |
| `m.unset_item=val` | `first` / `last` / `key` | Remove array item |
| `m.shuffle=yes` | None | Shuffle array items |
| `m.solve=yes` | None | Solve mathematical expression |
| `m.to_str=yes` / `m.to_int=yes` / `m.to_num=yes` / `m.to_bool=yes` | None | Cast data type |
| `m.append='str'` / `m.prepend='str'` | String literal | Append/prepend text |

---

## 5. Outputters Reference (`o.*`)
Outputters handle the output flow of a resolved and modified shortcode value.

| Outputter | Description |
| :--- | :--- |
| `o.die=yes` | Outputs the value and calls PHP `die()` |
| `o.echo=yes` | Echoes the value (retains the value for next outputters) |
| `o.dump=yes` | Calls PHP `var_dump()` on the value |
| `o.console=yes` | Logs the value in the browser web console (retains value) |
| `o.log=yes` | Logs the value in system error logs (retains value) |
| `o.destroy=yes` | Discards the output completely, returning `""` |
| `o.set='path'` | Saves value at environment path; returns `""` |
| `o.merge_with='path'` | Merges array value into target path array; returns `""` |

---

## 6. Escaping Methods Reference (`esc.*`)
Strictly for database queries. All `esc` tags expect a **path to a variable** (e.g. `user.age`), **not** a literal string.

| Escaper | Output Format | Example Query Reference |
| :--- | :--- | :--- |
| `[esc.int path /]` | Safe Integer | `post_id = [esc.int request.id /]` |
| `[esc.num path /]` | Safe Float / Numeric | `price = [esc.num request.price /]` |
| `[esc.table path /]` | Sanitised Table Name | `FROM [esc.table request.tbl /]` |
| `[esc.str path /]` | Single Quoted Escaped String | `WHERE name = [esc.str request.name /]` |
| `[esc.id path /]` | Single Quoted Sanitised ID | `WHERE code = [esc.id request.code /]` |
| `[esc.date path /]` | `DATE('escaped_date')` | `WHERE joined = [esc.date request.date /]` |
| `[esc.like path /]` | Escaped LIKE wildcards | `WHERE slug LIKE '%[esc.like req.slug /]%'` |
| `[esc.unsafe path /]` | Comma-delimited numbers | `WHERE id IN ([esc.unsafe req.ids /])` |
| `[esc.unsafe.quotes path /]`| Comma-delimited single quoted strings | `WHERE name IN ([esc.unsafe.quotes req.names /])` |
