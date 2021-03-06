﻿# ORM Lite

## Requirement

- **C++ 14** Support :wink:
  - MSVC >= 14 (2015)
  - gcc >= 5
- **SQLite 3** (zipped in *src*)

## Include to Your Project

``` C++
#include "ORMLite.h"
using namespace BOT_ORM;
```

## Inject Class

``` C++
#include "ORMLite.h"
using namespace BOT_ORM;

struct MyClass
{
    int id;
    double score;
    std::string name;

    // Inject ORM-Lite into this Class
    ORMAP (MyClass, id, score, name)
};
```

In this Sample, `ORMAP (MyClass, id, score, name)` means that:
- `Class MyClass` will be mapped into `TABLE MyClass`;
- `int id`, `double score` and `std::string name` will be mapped
  into `INT id`, `REAL score` and `TEXT name` respectively;
- The first item `id` will be set as the **Primary Key** of the Table;

Note that:
- `ORMAP (...)` will **auto** Inject some **private members**
  , but **NO damage** to the Class :wink:
- Currently Only Support
  - T such that `std::is_integral<T>::value == true`
  - T such that `std::is_floating_point<T>::value == true`
  - T such that `std::is_same<T, std::string>::value == true`
  - which are mapped as `INTEGER`, `REAL` and `TEXT` (SQLite3);
- Field Names MUST **NOT** be SQL Keywords;
- MyClass MUST be **Copy Constructible**;
- `std::string` Value MUST **NOT** contain `\0` (SQLite3 Constraint);

## ORMapper

### ORMapper\<MyClass\> (const std::string &dbName)

Construct a **O/R Mapper** in file `dbName`;

All operations on `MyClass` will be mapped into `dbName`;

### const std::string &ErrMsg () const

Return the latest Error Message;

### bool CreateTbl ([const Myclass &value])

Create Table `MyClass` for class `MyClass`;

Execute `CREATE TABLE MyClass (...);`

Remarks:
- Pass a value to Create Table
  if `Myclass` has **NO** Default Constructor

### bool DropTbl ()

Drop Table `MyClass` from the DB File;

Execute `DROP TABLE MyClass;`

### bool Insert (const MyClass &value)

Insert `value` into Table `MyClass`;

Execute `INSERT INTO MyClass VALUES (...);`

### bool Insert (const T\<MyClass\> &values)

Insert `values` into Table `MyClass`;

`values` must **SUPPORT** `forward_iterator`;
    
Execute `INSERT INTO MyClass VALUES (...), (...) ...;`

### bool Update (const MyClass &value)

Update Entry `value` in Table `MyClass`
with the Same `KEY` with variable `value`;

Execute `UPDATE MyClass SET (...) WHERE` `KEY` `=` `value.id` `;`

### bool Update (const T\<MyClass\> &values)
    
Update Entries with the Same `KEY` with `values`;

`values` must **SUPPORT** `forward_iterator`;

Execute Multiple `UPDATE MyClass SET (...) WHERE` `KEY` `=` `value.id` `;`

### bool Delete (const MyClass &value)

Delete Entry `value` in Table `MyClass`
with the Same `KEY` with variable `value`;

Execute `DELETE FROM MyClass WHERE` `KEY` `=` `value.id` `;`

### ORQuery Query (MyClass &queryHelper)

Return new `ORQuery` object and Capturing `queryHelper`;

Details in `## ORQuery` Section;

### Note that

Only a **bool returning** Function Succeeded,
it would return `true`; otherwise, return `false`;

## ORQuery

### ORQuery &Where (const Expr &expr)

Generate `WHERE (` `expr` `)`;

Details in `## Expr` Section;

### ORQuery &OrderBy (const T &property, bool isDecreasing = false)

Generate `ORDER BY` `field name of (property)` or `ORDER BY` `field name of (property)` `DESC` (if isDecrease);

Remarks:
- If `property` is not a member of `queryHelper`,
  throw `std::runtime_error`;

### ORQuery &Limit (size_t count, size_t offset = 0)

Generate `LIMIT` `count` `OFFSET` `offset`;

### std::vector\<MyClass\> ToVector () / std::list\<MyClass\> ToList ()

Retrieve Select Result under **Constraints**;

Execute `SELECT * FROM MyClass WHERE ... ORDER BY ... LIMIT ...;`

Remarks:
- Deserialize Value to `MyClass &queryHelper`;
- Copy `queryHelper` to `vector` / `list`;

### long Count ()

Return:
- Return Count under **_WHERE_ Constraints**;
- -1 if **Query Error**;

Execute `SELECT COUNT (*) FROM MyClass WHERE ... ORDER BY ... LIMIT ...;`

### bool Delete ()

Delete Entries under **_WHERE_ Constraints**;

Execute `DELETE FROM MyClass WHERE ... ORDER BY ... LIMIT ...;`

Return:
- `true` if Succeeded;
- `false` otherwise;
    
## Expr

### Expr (const T &property, const std::string &relOp = "=", T value = property)

Generate `field name of (property)` `relOp` `value`;

Remarks:
- `Expr (property)` is short for `Expr (property, "=", property)`
- If `property` is not a member of `queryHelper`,
  throw `std::runtime_error` at Calling `Where`;

### Expr operator && / || (const Expr &left, const Expr &right)

Generate `(` `left` `&& / ||` `right` `)`;

### struct Field_Expr\<T\> (const T &property) (*Helper Struct of Expr*)

#### Expr operator *OP* (T value)

Return `Expr { property, OP_STR, value }; }`;

| OP |  OP_STR  |
|----|----------|
| == |    "="   |
| != |   "!="   |
|  > |    ">"   |
| >= |   ">="   |
|  < |    "<"   |
| <= |   "<="   |

### Field_Expr<T> Field (T &property)

Return `Field_Expr<T> { property }`;