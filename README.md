---------
title: SQL String Pattern Matching Technique 
---------
author: Quang Huy Pham
---------
Disclaimer: This learning note is part of SQL Virtual Data Appreticeship offered by Danny Ma. I mostly follow along with his guidence but also adding my thought process in each technique/sections

Enjoy!!
Huy

# SQL String Pattern Matching Technique 
String pattern matching is tryly an essential skill for SQL. There are a few key tools and tricks that we have up our sleeve for any text matching problem:
- Exact pattern matching with `=`
- Efficient fuzzy matching using `LIKE` for case sensitive and `ILIKE` for case insensitive
- Regular Expressions or RegEx Using the `-` for case sensitive and `-*` for case insensitive matching
- Using `NOT LIKE`, `NOT ILIKE`, `!-` and `!-*` for negative matches

## Exact Pattern Matching

Use `=` in our `WHERE` filters and `CASE WHEN` statements to perform this exact matching
THe only thing to be aware of is that you need to hav the text eactly correct including the lettercase, spaces and any punctuation!


```sql
WITH test_date (text_value) AS (
  VALUES
    ('Hello World!'),
    ('hello World!'),
    ('Hello to the world!'),
    ('Hello, world')
)
SELECT
  text_value
FROM test_date
WHERE text_value = 'Hello World!';
```


<details> 
<summary> 
Click here to show the output 
</summary>

| text_value   |
| ------------ |
| Hello World! |
</details>

## Case Sensitive Fuzzy Matching
Fuzzy matching refers to any sort of matching which is not straight exact matches.

A wildcard percentage sign `'%'` character can be used to replace any number of characters in a string field and an underscore wildcard `'_'` can be used to replace any single character

### Wildcard Examples
```sql
WITH test_date (text_value) AS (
  VALUES
    ('Hello World!'),
    ('hello World!'),
    ('Hello to the world!'),
    ('Hello, world')
)
SELECT text_value
FROM test_date
WHERE text_value LIKE 'Hello%';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value          |
| ------------------- |
| Hello World!        |
| Hello to the world! |
| Hello, world        |
</details>

```sql
WITH test_date (text_value) AS (
  VALUES
    ('Hello World!'),
    ('hello World!'),
    ('Hello world!')
)
SELECT text_value
FROM test_date
WHERE text_value LIKE '%World!';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value   |
| ------------ |
| Hello World! |
| hello World! |
</details>

- Multiple Wildcard example
```sql
WITH test_date (text_value) AS (
  VALUES
    ('Hello World!'),
    ('hello World!'),
    ('Hello world!'),
    ('Will this hello World show up?')
)
SELECT text_value
FROM test_date
WHERE text_value LIKE '%el%Wor%';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value                     |
| ------------------------------ |
| Hello World!                   |
| hello World!                   |
| Will this hello World show up? |
</details>

- `'_'` Wildcards example
```sql
WITH test_date (text_value) AS (
  VALUES
    ('Hello World!'),
    ('helloWorld!'),
    ('Hello world!'),
    ('Jello World')
)
SELECT text_value
FROM test_date
WHERE text_value LIKE '_ello_World_';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value                     |
| ------------------------------ |
| Hello World!                   |
</details>

### Wrong Letter Case Scenario
The text that is accompanying the wildcards must be of the correst case also!
```sql
WITH test_date (text_value) AS (
  VALUES
    ('Hello World!'),
    ('hello World!'),
    ('Hello world!!!!'),
    ('HEllo World')
)
SELECT text_value
FROM test_date
WHERE text_value LIKE '_ello_world%';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value      |
| --------------- |
| Hello World!!!! |
</details>

## Case Insensistive Match
`ILIKE` is used when you need a case insensitive match where it doesn't matter if the string has upper case or lower case and you are only interest in the characters themselves

Tt's essentially the same as `LIKE` but should be used thoughtfully as there is an implicit cost in check text strings for different lettercases 0 it'sn not going to be as performant as checking for explicit lowercase or uppercase chracters when used with the `LIKE` operator

```sql
WITH test_date (text_value) AS (
  VALUES
    ('Hello World!'),
    ('hello World!'),
    ('Hello world!!!!'),
    ('HEllo World'),
    ('HeLlO WoRlD')
)
SELECT text_value
FROM test_date
WHERE text_value ILIKE '%hello world%';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value      |
| --------------- |
| Hello World!    |
| hello World!    |
| Hello world!!!! |
| HEllo World     |
| HeLlO WoRlD     |
</details>

> In the `dvd_rentals.film_list` table - how many films have the word `unbelieveable` in the `description` field?

```sql
SELECT COUNT(*)
FROM dvd_rentals.film_list
WHERE description ILIKE '%unbelieveable%';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| count           |
| --------------- |
| 45              |
</details>

### `NOT LIKE` and `NOT ILIKE`
We can also use `NOT` in front of `LIKE` and `ILIKE` as a negative pattern matcher

> Find all the films without the world 'monkey' in the description

```sql
SELECT COUNT(*)
FROM dvd_rentals.film_list
WHERE description NOT ILIKE '%monkey%';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| count           |
| --------------- |
| 911             |
</details>

A note on all of these fuzzy matching functions - you can use them with `AND` and `OR` operators for more complex logic that includes multiple words or phrases that you need to match

```sql
SELECT COUNT(*)
FROM dvd_rentals.film_list
WHERE 
  description NOT ILIKE '%monkey%'
  AND description ILIKE '%unbelieveable%';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| count           |
| --------------- |
| 41              |
</details>

## Regular Expressions
Regular Expressions or regex are the final piece of the pattern matching puzzle and VERY VERY useful but can be VERY tricky when you get started

Some interesting scenarios where regex is commonly deployed include:
- Making sure passwords are strong enough
- Identifying whether an email address is valid
- Validating credit card number inputs from different porviders (Visa, Amex, etc)
- Checking for valid address inputs
- Making sure IP addresses are in the right format
- Validating URLs
- Checking for valid bitcoin addresses

And for some more regular use cases in SQL:
- Identifying specific patterns in a long string to identify customers
- Seaarching for ultiple date formats in a free-text field
- Find a replacing multiple substrings without nested `SUBSTRING` functions
- Identifying mached patterns and re-inserting them to create a new text field

### Meta Characters
| Meta Characters | Description                                                      |
| --------------- | ---------------------------------------------------------------- |
| `|`               | Boolean `OR`                                                       |
| `*`              | Repeat the previous item `zero or more` times                      |
| `+`               | Repeat the previoius item `one or more` times                     |
| `? `              | Repeat the previous item `zero or one` time                        |
| `{m}`            | Repeat the previous item `exactly m `times                         |
| `{m,}`            | Repeat the previous item `m or more` times                         |
| `{m,n}`           | Repeat the previous item `at least m` and `not more than n` times    |
| `.`               | Wildcard to denote any chracter                                  |
| `^`               | Refers to the beginning of the text input                        |
| `$`               | Refers to the end of the text input                              |
| Parentheses `()`  | Used to group items into a single logical item                   |
| Breaket `[]`      | specifies a character class, just as in POSIX regular expression |

## RegEx Example
> Keep only records which **START** with `hello` or `Hello` or a capital `J`

```sql
WITH test_data (text_value) AS (
VALUES
  ('Hello World!'),
  ('hello World!!'),
  ('Hello world!!!!!'),
  ('HELLO world'),
  ('Peanut Butter Jelly Sandwich'),
  ('Just for fun!'),
  ('just kidding!')
)
SELECT text_value
FROM test_data
WHERE text_value ~ '^(Hello|hello|J)';
-- Using ^ to signal to match the start, then Parentheses `()` and | to specify option that the text input can start with. In this case Hello, hello, J is valid
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value       |
| ---------------- |
| Hello World!     |
| hello World!!    |
| Hello world!!!!! |
| Just for fun!    |
</details>

> Keep records that end in 2 to 5 exclaimation marks

```sql
WITH test_data (text_value) AS (
VALUES
  ('Hello World!'),
  ('hello World!!'),
  ('Hello world!!!!!'),
  ('HELLO world'),
  ('Peanut Butter Jelly Sandwich')
)
SELECT text_value
from test_data
WHERE text_value ~ '^.*(!){2,5}$';
-- Using ^ to signal beginning of the text, then .* to replace any number of character until the end of the string (!){2,5} signal how many time '!' can be repeated, then end of the string with $
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value       |
| ---------------- |
| hello World!!    |
| Hello world!!!!! |
</details>

> Keep records with 2 or more lowercase letter `l`'s next to each other
```sql
WITH test_data (text_value) AS (
VALUES
  ('Hello World!'),
  ('hello World!!'),
  ('Helllllllo world!!!!!'),
  ('HELLO world'),
  ('Peanut Butter Jelly Sandwich')
)
SELECT
  text_value
FROM test_data
WHERE text_value ~ '^.*(l){2,}.*$';
-- Đơn giản :P
```
> How many movie titles start with `A`?
```sql
SELECT
  COUNT(title) as movie_count
FROM dvd_rentals.film_list
WHERE title ~ '^A';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| movie_count           |
| --------------- |
| 46              |
</details>

> Which movies have 2 words in the title that both begin with the letter M

```sql
SELECT title
FROM dvd_rentals.film_list
WHERE title ~ '^(M\w+\sM\w+$)';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| title          |
| -------------- |
| MAGIC MALLRATS |
| MAUDE MOD      |
| MILE MULAN     |
| MULAN MOON     |
| MUPPET MILE    |
</details>

> Which movies have 2 words in the title that both begin with the same letter
```sql
SELECT title
FROM dvd_rentals.film
WHERE title ~ '^(\w)\w+\s\1\w+$'
-- Using (\w) to group the first letter at the beginning and \1 to refer to this first letter group to find exact matching word
LIMIT 10;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| title               |
| ------------------- |
| BIKINI BORROWERS    |
| BLANKET BEVERLY     |
| BOONDOCK BALLROOM   |
| BORROWERS BEDAZZLED |
| BROTHERHOOD BLANKET |
| BUCKET BROTHERHOOD  |
| CAT CONEHEADS       |
| CHARIOTS CONSPIRACY |
| CHEAPER CLYDE       |
| CONFUSED CANDLES    |
</details>

> Return movie names that have a title with the first word starting with A but with the second word not starting with letters D, M, A

```sql
SELECT title
FROM dvd_rentals.film
WHERE title ~ '^A\w+\s[^DMA]\w+$'
-- Using (\w) to group the first letter at the beginning and \1 to refer to this first letter group to find exact matching word
LIMIT 10;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| title            |
| ---------------- |
| ACE GOLDFINGER   |
| ADAPTATION HOLES |
| AFFAIR PREJUDICE |
| AFRICAN EGG      |
| AGENT TRUMAN     |
| AIRPLANE SIERRA  |
| AIRPORT POLLOCK  |
| ALADDIN CALENDAR |
| ALAMO VIDEOTAPE  |
| ALASKA PHANTOM   |
</details>

## Hierarchy of Preference
1. Use `LIKE` whereever possible as it is the most efficient and simple to read/understand
2. Use `ILIKE` sparingly as it is more computationally expensive than a case sensitive match
3. When using ReGEX `~` always try to left anchor `^` your patterns from the beginning of the text so valid indexes can be used
4. Avoid using SIMIAR TO which is available in most Standard SQL (word on the street is that this is going to be deprecated soon!)

`LIKE`, `ILIKE`, `~` with a left anchor `^` can make use of PostgreSQL column indexes -> speed up these operations on huge datasets

```sql
/* 
However the indexes must be created correly and use the 3 specific operator classes of `text_pattern_ops`, `varchar_pattern_ops` or `bpchar_pattern_ops` for their respentive index columns so B-Tree indexes can be created on text, varchar and char fields accordingly
*/
```

### Find and Replace
#### Simple Replace

```WITH test_data AS (
  SELECT 'Hello World!' AS text_value
)
SELECT
  REPLACE(text_value, 'Hello', 'Bonjour')
FROM test_data;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| text_value               |
| ------------------- |
| Bonjour World!    |
</details>

> Swap `Robot` with `Huge Polar Bear` in the description from the film called `ANGELS LIFE`

```sql
SELECT
  REPLACE(description, 'Robot', 'Huge Polar Bear') as new_description
FROM dvd_rentals.film
WHERE title = 'ANGELS LIFE';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| new_description |
| ------------------- |
| A Thoughtful Display of a Woman And a Astronaut who must Battle a Huge Polar Bear in Berlin    |
</details>

### Regular Expression Replace
Using REGEXP_REPLACE() to do this!

> Swap everthing from the beginning of the word `Woman` till the end of the word `Astronaunt` in the `description` column with `Weird Turtle` from the film called `ANGELS LIFE`

```sql
SELECT
  REGEXP_REPLACE(
    description,        -- Target text column
    'Woman.*Astronaut', -- What we want to find
    'Weird Turtle'      -- What we want to replace
    ) as new_description
FROM dvd_rentals.film
WHERE title = 'ANGELS LIFE';
```
<details> 
<summary> 
Click here to show the output 
</summary>

| new_description |
| ------------------- |
| A Thoughtful Display of a Weird Turtle who must Battle a Robot in Berlin |
</details>

> Using REGEXP_REPLACE to do product code padding
```sql
WITH products (product_code) AS (
VALUES
  ('1234-BOX'),
  ('3421-EACH'),
  ('35895-PACK'),
  ('451884-CARTON')
)
SELECT
  -- we still need the LPAD function to pad our 0's
  LPAD(
    -- replace anything comes from the dash '-' to the end with a blank string
    REGEXP_REPLACE(
      product_code,
      '-.*$',
      ''
    ),
    6,
    '0'
  ) AS product_id
FROM products;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| product_id |
| ------------------- |
| 001234 |
| 003421 |
| 035895 |
| 451884 |
</details>

`REGEXP_REPLACE` also have a optional `mode` parameter after the replacement. By deault when we appl the REGEXP_REPLACE, only the first instance of the matched pattern is replaced. We can alter this by specifying `'g'` after our  replacement string to specifiy global mofe

Example:
```sql
WITH test_data (text_value) AS (
  VALUES
  ('Say a little hello to my little friend, Danny!')
)
SELECT
  REGEXP_REPLACE(
      text_value,
      'little',
      'big'
  ) AS default_replace,
  REGEXP_REPLACE(
      text_value,
      'little',
      'big',
      'g'
  ) AS global_replace
FROM test_data;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| default_replace                             | global_replace                           |
| ------------------------------------------- | ---------------------------------------- |
| Say a big hello to my little friend, Danny! | Say a big hello to my big friend, Danny! |
</details>

We can use the Regex pattern groups we've done earlier to capture and reinsert those pattern group into our replacement string

Example:
```sql
WITH test_data(text_value) AS (
  VALUES 
    ('My friends'' names are Ken, Eric and Ben!')
)
SELECT
  REGEXP_REPLACE(
    text_value,
    '(\w+),\s(\w+) and (\w+)',
    'Mr. \1, Mr. \2 and Mr. \3'
  ) AS groups_usage
FROM test_data;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| groups_usage                             |
|---------------------------------------- |
| My friends’ names are: Mr. Ken, Mr Eric and Mr Ben! |
</details>

### Regex Match
We might be interested in retuning a result when our regular expression matches something in a target string instead of just returning a booklean true or false value

`REGEXP_MATCH` help to return a result when REGEX matches something in a target string instead of just retuning a boolean true or false value

The output for the `REGEXP_MATCH` is an array datatype
If we want to return only the first match, we can add `[1]` after the function expression

```sql
WITH test_data (text_value) AS (
  VALUES ('Hello World ABC123 XYZ123')
)
SELECT
  REGEXP_MATCH(text_value, '[A-Z]{3}[0-9]{3}')[1] AS first_match
FROM test_data;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| first_match                             |
|---------------------------------------- |
| ABC123 |
</details>

If we remove the `[1]` to specify the first element of the array output, we would get an array returned instead of a text column.

```sql
WITH test_data (text_value) AS (
  VALUES ('Hello World ABC123 XYZ123')
)
SELECT
  REGEXP_MATCH(text_value, '[A-Z]{3}[0-9]{3}') AS array_output
FROM test_data;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| array_output                             |
|---------------------------------------- |
| ["ABC123"] |
</details>

However, as we might expect, the array should have two elements `ABC123` and `XYZ123`, it only return `ABC123`. Seems like `REGEXP_MATCH()` only find the first match in the string.

I have also tried adding `'g'` global parameter but the function does not support this. Interesting!

Moving on to the next one...

We can also specify groups within the regex to retun multiple elements in the array like so - let's say we wanted to split our regex into 2 groups of `([A-Z]{3})` and `([0-9]{3})`

We can then further decompose our array output by using the UNNEST function to opo each element of the array into it's own row - we call this transformation `flattening` an array with multiple elements to multiple rows

```sql
WITH test_data(text_value) AS (
  VALUES('Hello World ABC123 XYZ123')
)
SELECT
  UNNEST(REGEXP_MATCH(
    text_value,
    '([A-Z]{3})([0-9]{3})'
  )) AS flat_output
FROM test_data
```
<details> 
<summary> 
Click here to show the output 
</summary>

| flat_output                             |
|---------------------------------------- |
| ABC |
| 123 |
</details>

A note when dealling with `UNNEST` is that there will be duplicate rows of data when you have other columns that are being selected along with the flatten array!

### Multiple Regex Matches

So remember the previous `REGEXP_MATCH` we use previously to find string pattern and return the first match, it doesn't support global...

But guess what, we do have the version to match multiple similar string pattern, which is `REGEXP_MATCHES`. And I learn that this one can use `'g'` parameter :)

`REGEXP_MATCHES` example:
```sql
WITH test_data(text_value) AS (
  VALUES('Hello World ABC123 XYZ123')
)
SELECT
  (REGEXP_MATCHES(text_value, '[A-Z]{3}[0-9]{3}'))[1] AS default_mode,
  (REGEXP_MATCHES(text_value, '[A-Z]{3}[0-9]{3}','g'))[1] AS global_mode
  -- Unlike what I expected, we still use [1] to extract different matches, turn out that for each match, it return into a new row output, each array return (if we omit [1]) represent one single match only. Interesting!
FROM test_data;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| default_mode                             | global_mode                           |
| ------------------------------------------- | ---------------------------------------- |
| ABC123 | ABC123 |
| *null*| XYZ123 |
</details>

### Addres Type Example
> What is the count of address type in the dvd_rentals.address table

Let's quickly review this dataset to answer the question given above
```sql
SELECT address
FROM dvd_rentals.address
LIMIT 5;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| address              |
| -------------------- |
| 47 MySakila Drive    |
| 28 MySQL Boulevard   |
| 23 Workhaven Lane    |
| 1411 Lillydale Drive |
| 1913 Hanoi Way       |
</details>

We can spot out that the end of each string ('Drive', 'Boulevard', etc) indicate the type of address that we're looking for. We can use `REGEXP_MATCH()` to answer this question

```sql
SELECT
  (REGEXP_MATCH(address, '\w+$'))[1] AS address_type,
  COUNT(*) AS address_type_count
FROM dvd_rentals.address
GROUP BY 1
ORDER BY 2 DESC;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| address_type | address_type_count |
| ------------ | ------------------ |
| Parkway      | 76                 |
| Manor        | 66                 |
| Lane         | 60                 |
| Street       | 60                 |
| Place        | 59                 |
| Avenue       | 59                 |
| Way          | 59                 |
| Drive        | 56                 |
| Loop         | 54                 |
| Boulevard    | 54                 |
</details>

### Another Padding Exaple
Let's use REGEXP_MATCH to also carry out the padding example we've done earlier

```sql
WITH products (product_code) AS (
VALUES
  ('1234-BOX'),
  ('3421-EACH'),
  ('35895-PACK'),
  ('451884-CARTON')
)
SELECT
  LPAD(
    (REGEXP_MATCH(product_code, '^\d{1,6}'))[1],
    6,
    '0'
  ) AS padded_product_id
FROM products;
```
<details> 
<summary> 
Click here to show the output 
</summary>

| padded_product_id |
| ----------------- |
| 001234              |
| 003421              |
| 035895             |
| 451884            |
</details>

### Split String
We can use the `SPLIT_PART` to seperate these values by specifying a delimiter for each of these values as well as a position for the new columns

Example
```sql
WITH events (logs) AS (
VALUES
  ('user1-click-20210101'),
  ('user2-buy-20210201')
)
SELECT 
  SPLIT_PART(logs, '-',1) AS user_id,
  SPLIT_PART(logs, '-',2) AS event_type,
  SPLIT_PART(logs, '-',3)::DATE AS event_date
FROM events;
```
<details> 
<summary> 
Click here to show the output 
</summary>
| user_id | event_type | event_date |
| ------- | ---------- | ---------- |
| user1   | click      | 2021-01-01 |
| user2   | buy        | 2021-02-01 |
</details>

### Summary

To summarize, we've gone through all basic to immediate (REGEX) String Transformations in SQL. 

#### Basic String functions
1. We learnt how to use `LEFT`, `RIGHT` to extract first & last N characters
2. We learnt `SUBSTRING` to get substring at the middle of the text
3. `CHAR_LENGTH` to count length of string
4. `POSITION` to find first position of a simple string in a text
5. `LOWER`, `UPPER`, `INITCAP` to convert string to lower, UPPER and Title case
6. Format string with `TO_CHAR`
7. Padding with `LPAD` & `RPAD`

#### Pattern Matching

1. `LIKE` & `ILIKE` with wildcard `'%'` and `'_'` to search for simple string pattern
2. `NOT LIKE` & `NOT ILIKE` as a negative pattern matcher
3. REGEX (So so interesting and powerful)
  This deserve a topic on there own 
  3.1. `~` to find string pattern
  3.2. `REGEXP_REPLACE` to replace a substring that match a string pattern
  3.3. `REGEXP_MATCH` and `REGEXP_MATCHES` to return the matching substring pattern.
  Note that `REPLACE` and `MATCHES` function have `'g'` global parameters so we can replace and match many similar string pattern within one string.

  And many many more :)

I won't share any cheatsheet since I'm still a very beginner and having the cheatsheet alongside doesn't neccessarily benefit beginner learning recall at all in my opinion.

Happy learning !
Huy~
