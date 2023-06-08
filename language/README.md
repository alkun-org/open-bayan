# Bayan Language

There are many proven open standards available out there (OpenAPI, Json-schema, RAML, Cue), however most of them are too verbose, ambiguous and somewhat complicated. After many iterations, we came up with a clean and concise DSL that average developer can quickly graps the concept called Al-Bayan (or just Bayan for simplicity).

*This specification will be frozen by July 2023*

### The language
Bayan is a 2 part languages. First part is a compact **Bayan DSL** for us human and second part is a **Bayan JSON Schema** compiled from the former and optimized for computer processing. Now let's dive into these 2 formats.

### Shared Rules and Conventions
1. Bayan is case sensitive.

2. Bayan **Keywords** is in all uppercase.

3. We recommended to use **camelCase** for everything else.

4. Every names/properties beside the reserved keywords can be override.

5. Use tab or 4 spaces for indentation for clarity.

6. Please follow the rules here when creating your schema, do not do ambiguous, hackish things or add extensions. It is important as this specification will be permanently frozen and we don't want a fragmentation.

## Bayan JSON Schema

### Document Type
This is a valid JSON documents with "*.json*" extension. For optimization and compression, we also use CBOR serialization format (We use it for some *Box*, our own archive specification). 

### Reserved Keywords
TIP, TYPE, NULLABLE, DEFAULT, REQUIRED, SKIP, LOOSE, SCHEMA,
NULL, STR, NUM, BOOL, OBJ, URL, DATE, TIME, ENUM, CONST,
MIX, MAX, MINLEN, MAXLEN, ARRAY, PATTERN

1. **TIP (string)** - an inline comment for current object.

2. **TYPE (string)** - to define current object's data type. Eg: `"TYPE": "STR"`

3. **NULLABLE (boolean)** - Boolean "true" tells validator this value accepts a *null*.

4. **DEFAULT** - Tells validator to use this value if not provided.

5. **REQUIRED (string[])** - Property that must be included, no matter if values is empty or null. Use sub-array for either one of the property, eg: `"REQUIRED": ["name", ["age", "dob"], "hobby"]`, here *name* and *hobby* are mandatory, but for *age* and *dob* can be either one of them or both.

6. **SKIP (string[])** - Tells validator to skip or ignore this properties.

7. **LOOSE (boolean)** - Allow an object to have additional properties not defined inside schema.

8. **ARB (string[[regex, string]])** - Allow pattern matching on arbitrary properties, eg:
```
"myProp": { "TYPE": "STR" }
"ARB": [["^\w+$", "myProp"]]
````

9. **SCHEMA (string)** - URL or anchor to bayan object for validation purposes, eg: `"SCHEMA": "#person"`.

10. **NULL/STR/NUM/BOOL/OBJ** - Use in defining a valid JSON datatypes type, eg: `"TYPE": "NUM"`.

11. **URL (string)** - a custom datatype for URL.

12. **DATE (string)** - a custom datatype for a date /datetime in ISO 8601 format.

13. **TIME (string)** - a custom datatype for a time in format  `hh:mm:ss[Z|(+|-)hh:mm]`.

14. **ENUM (string[]/number[])** - a custom datatype for an array of fixed values / enumerates, eg: `"ENUM": ["hello", "world"]`. Default type is `"TYPE": "ENUM"`. Array cannot mix types.

15. **CONST (string/number/bool)** - define a fixed value, eg: `"CONST": "hello"`. Default type is `"TYPE": "CONST"`.

16. **MIN/MAX (number)** - If type is STR, this will limit the string value, if numbers it will limit the number's range inclusive, eg: `"MIN": 10`. 

17. **MINLEN/MAXLEN (number)** - Limit an array length inclusive. 

18. **ARRAY (boolean)** - To tell this property is an array, use together with MINLEN/MAXLEN.

19. **PATTERN (regex)** - To allow particular string patterns only, eg: `"TYPE": "STR", "PATTERN": "^[A-Za-z_]+$"`.


Scroll down for details on validation and example.


## Bayan DSL

Bayan DSL (DSL from here on) is a compact language to make schema or API more readable and less stress. Most important rule of DSL is no ambiguity, magic or side effect, every statement must be clear and agreeable.


### Document Type
It is highly recommended though not mandatory to use "*.bayan*" extension.

### Reserved Keywords
All of the above with addition of IMPORT and EXTEND.

### Rules and Conventions
1. Every line is a single property definition unless it's inside **grouping** clause which allows to span multiple lines.

2. Definition can be separated by a line break **\n** or semi-colon.

3. First level properties will be assign to Root object.

4. Comma as items separator is not needed, the parser should omit them.


### Comments and Tips
Comment begin with double slash "//" and until line break.
Comment can be an entire line or at end of the definition.

Eg: `name = "Abdul" // your display name`

Tip is similar to comment but instead of strip upon parsing, it will be add to current object for reference.

Eg: `name STR # name is a string`
will produce `"name":{ "TYPE": "STR", "TIP": "name is a string"}`.

### Multiline support
Single line definition/comment/tip (except for double-quoted string which is always multiline) can be expanded to multiline by adding space + backslash at end of line. Eg:

```
// This is a first line, \
   while this is the next line \
   and this is the last line
```

### Property Definition
This is the most significant feature of the DSL.

`member STR[:10] <2:32>`

Line above translated to JSON below

`"member": { "TYPE":"STR", "MAXLEN":10, "ARRAY": true, "MIN":2, "MAX":32 }`

as you can see, it's very compact.

### Property Type
Each property must have a type, if omitted it will be considered as an **object** except for arrays.

### Limiting Ranges
"<", ">", "[", "]" is a special grouping symbol, it is only use to limit a string, number or an array.

`<5:10>` will produce **MIN** and **MAX** as explained above. Use **colon** ":" as a separator. If single value given eg: `<5>` considered as **MIN** only, while `<:10>` is **MAX** only.

`[5:10]` is similar to above but for an array, will produce **MINLEN** and **MAXLEN** instead.


### Property assignment and Default value
Use equal symbol "=" to replace value of property, eg: `name = "Abdul"`

Use ":" at end of definition to add a default value, eg: `name STR<2:32> : "Abdul"`

### Defining a constant
Use colon with equal symbol ":=" to define a constant value, eg: `name := "Abdul"`

### Multiple nullable declaration
Using DSL you can do `NULLABLE talent & hobby` instead of true, which will inject `"NULLABLE": true` to the given properties. 

### OBJECT and TUPLE
Object use for grouping multiple properties together. Keep in mind that **Object and Tuple are immutable (don't change) upon creation**.

Eg: `media = {mediaType STR; url URL}`

Tuple is a set of constant values or normally refer as enumeration.

Eg: `status = ("pending" "approved" "suspend")`


### Extending Object
Since object is immutable, you cannot modify their properties. The only way is to EXTEND them. Extend will **shallow copy** that object and create a new object with modified properties. You can extend multiple by chaining them using "&" symbol.

Eg 1: `image extend media & { width NUM; height NUM}`

Example above, we extend media object and add 2 new properties.


### Import external DSL
We can import definitions from another bayan document, all we need is the URL.

Eg: `IMPORT ct "https://www.alkun.org/bayan/thing.bayan"`

In example above, we load bayan from the URL (string), transform it and assign it as ct object in our root like this `this.root.ct = {...}`. Then we can use it like this `creator = ct.thing`

We can also import directly into root like this `IMPORT "https://www.alkun.org/bayan/thing.bayan"`


## Parsing and Validation
Bayan DSL is a top down language. Everything must be in proper sequence (create property above, refer to property below). This way we can easily implement a fast single pass parser.


## Example
Here some example of Bayan DSL and the translated Bayan JSON.

Example 1:
```
# Example of schema definition
prayerNames = ("Fajr" "Dhuhr" "Asr" "Maghrib" "Isha")
person = {
    givenName STR
    familyName STR
}

schema {
    imam EXTEND person & {
        schedule ENUM prayerNames []
    }
    REQUIRED imam
}

testSchema { // we create a json data to test schema above
    imam = {
        givenName = "Abdul"
        familyName = "Mutallib"
        schedule = ("Fajr" "Maghrib" "Isha")
    }
}
```

```
{
    "TIP": "Example of schema definition",
    "prayerNames": ["Fajr", "Dhuhr", "Asr", "Maghrib", "Isha"],
    "person" : {
        "givenName": { "TYPE": "STR" },
        "familyName": { "TYPE": "STR" }
    },

    "schema": {
        "imam": {
            "givenName": { "TYPE": "STR" },
            "familyName": { "TYPE": "STR" },
            "schedule": {
                "TYPE": "ENUM",
                "ENUM": ["Fajr", "Dhuhr", "Asr", "Maghrib", "Isha"],
                "ARRAY": true
            }
        },
        "REQUIRED": ["imam"]
    },

    "testSchema": {
        "imam": {
            "givenName": "Abdul",
            "familyName": "Mutallib",
            "schedule": ["Fajr", "Maghrib", "Isha"]
        }    
    }
}
```
As you can see, how compact and readable our Bayan DSL is compared to the JSON based schema.


Example 2: Importing another Bayan DSL
```
IMPORT cw "https://alkun.org/bayan/creative-work"
book = {
    publisher EXTEND cw.publisher
    # Here we add properties from creative-work \
    directly as book properties
}
```

```
{
    "book": {
        "publisher": {
            "name": { "TYPE": "STR", "MIN": 2, "MAX": 32 },
            ...
        },
        "TIP": "Here we add properties from creative-work directly as book properties"
    }
}
```

Example 3: Required one of the properties:
```
contactInfo = {
    name STR
    email STR : "spam@mydomain.com"
    phone STR
    REQUIRED name & email | phone
    # Either phone or email must be given here. When none, \
    will also take Default value if available
}
```

```
{
    "contactInfo": {
        "name": { "TYPE": "STR" },
        "email": { "TYPE": "STR", "DEFAULT": "spam@mydomain.com" },
        "phone": { "TYPE": "STR" },
        "REQUIRED": ["name", ["email", "phone"]],
        "TIP": "Either phone or email must be given here. When none, will also take Default value if available"
    }
}
```

Example 4: Matching properties:
```
house = {
    color ENUM ("Red" "Green" "Blue")
    count NUM <:10>
    ARB /^\w+Color$/ color  // validate "roofColor": "Blue"
    ARB /^\w+Count$/ count
}
```

```
{
    "house": {
        "color": {
            "TYPE": "ENUM",
            "ENUM": ["Red", "Green", "Blue"]
        },
        "count": { "TYPE": "NUM", "MAX": 10 },
        "ARB": [
            ["^\\w+Color$", "color"]
            ["^\\w+Count$", "count"]
        ]
    }
}
```

## Reference Implementation

This specification was designed so it's easy to compose a schema by hand. You can see a reference implementation in ES6 [here](https://github.com/alkun-org/open-bayan/blob/main/language/javascript/parser.js).

Take a look at this [common-type.bayan](https://github.com/alkun-org/open-bayan/blob/main/project/alkun-org/entity/common-type.bayan) and the JSON generated using tool above [common-type.json](https://gist.github.com/alkun-org/017618d845a4f6850e1da48bd9cfe56f).
