# pmMapper

The pmMapper helps to map an existing URL to a different URL, by transforming the query string (url parameters).

## URL query string

URL parameters must be structured as 

```
param1=value1[,value2...]&param2=value1[,value2...]...
```

No special characters are allowed.
Any parameter can have multiple values, seperated by a coma.
If a parameter apears multiple times, it will be treated as a list of values for the same parameter.

## Simple example

```
Input
foo=bar&prod=test
```

```
Mapping Rule
[{ foo : 'bar'} , { test : ' pass', day : 'fri'}]
```

```
Output
test=pass&day=fri
```

## Data model

The pmMapper controls three data-collections.

- Input params  (the query string to be mapped, every request provides params that will be parsed)
- Output params (the resulting query string, with every mapping process, a new output param object will be created)
- Mapping rules (rules are inserted before mapping and will be applied during the mapping process)

## Mapping process

The mapping process parses the input params and creates an empty output params object.
The process will then apply every mapping rule.

The process returns the populated output params object.

## Mapping Rule

A mapping rule can have two or three components, provided as objects in an array

```
{condition object} , { action object} [,{ else action object}] 
```

The else action can be omitted.


### Condition Object

Unless otherwise specified, all conditions are checked against the input params.

A single string is generally treated like an array with only a single string.

As any parameter can have multiple values, a check for a single value returns true, if the value-list contains the value.
(Example months=jan,feb,may returns true for months : feb)

By default, all conditions are checked under the OR-assumption.  With the $and-operator it can be changed to AND.

```
{}  // Empty condition, always true.
{foo : 'bar'}  or {foo : ['bar']} // true for param 'foo' with a value 'bar'
{foo : ['bar','box']}  // true for param 'foo' with value 'bar' OR 'box'
{foo : 'bar', prod : 'test'}  // true for param 'foo' with 'bar' OR 'prod' with 'test'
```


```
{ $or :  { remaining condition } }  // remaining condition will be under OR-assumption (default behavior)
{ $and:  { remaining condition } }  // remaining condition will be under AND-assumption

{ $and :  { day : 'sun', prod : 'test'}}  // true for param 'day' with 'sun' AND 'prod' with 'test'
{ $and :  { day : ['sun','mon']}  // true for param 'day' with value 'sun' AND 'mon'

```

## Action object

Unless otherwise specified, all actions are done to the output params.

A single string is generally treated like an array with only a single string.

```
{}  // Empty action, does nothing.
{foo : 'bar'}  or {foo : ['bar']} // adds 'bar' to the param 'foo'
{foo : ['bar','box']}  // adds 'bar' AND 'box' to the param 'foo'
{foo : 'bar', prod : 'test'}  // adds 'bar' to 'foo' and 'test' to 'prod'
```

```
{ $copy : true} or { $copy : '*' }  // copies all params from the input to the output
{ $copy : 'foo'} or { $copy : ['foo'] }  // copies the param 'foo' from the input to the output
{ $clear : true} or { $clear : '*' }  // deletes all params from the output
{ $clear : 'foo'} or { $clear : ['foo'] }  // deletes the param 'foo' from the output
```

## Short notation

All mapping rules are difined in condition and action objects.  For convinience some mapping rules can be difined in strings.
(Elements within the string will be trimmed.)

```
Same mapping rule
[{ foo : 'bar' }, { test : 'pass' }]  // default syntax as objects
[ ' foo : bar ' , ' test : pass ' ]  // using two strings
' foo : bar ; test : pass '  // using one string, no array needed
```

- String is split at semi-colon (;) for condition and action object.
- For condition and action, string is split at colon (:)
- If even items, one object is created: { item0 : 'item1', item2 : 'item3' ...}
- If odd items, one object is created with a nested object:  { item0 : { item1 : 'item2', item3 : 'item4'...}}
