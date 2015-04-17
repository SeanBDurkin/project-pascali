# literal integers #
```
literal base 10 integer = [s , "-"] , digit , {digit};
literal base 16 integer = [s , "-"] , "$" , hex , {hex};
literal base X integer = [s , "-"] , "$$" , digit , [digit] , "/" , alphanumeric , {alphanumeric};
(* The two digits in the above specify the base which may range from 2..36 . *)
literal integer = literal base 10 integer | literal base 16 integer | literal base X integer;
(* Literal integers may not exceed the range -2^127 .. (2^128)-1 *)
```

## Examples ##
Here are two literal integers
```
-00568989
$AF
```

# floats #
```
normal literal float = literal base 10 integer , [ "." , digit , {digit}] ,
              [("e"|"E") , ["-"] , digit , {digit}];
NaN = "NaN";
INF = "INF";
negative INF = s , "-INF";
literal float = normal literal float | NaN | INF | negative INF;
```


# binary coded decimal values #
```
literal bcd = literal base 10 integer , [ "." , digit , {digit}];
```

# boolean #
```
literal False = "False";
literal True = "True";
literal boolean = literal False | literal True;
```

# literal characters #
```
eoln = ? end-of-line mark or new-line characater. ?;
literal simple char = "'" , (char - (eoln | "'")) , "'";
literal apos char = "''''";
literal utf-8 char = "#" , hex , hex;
literal utf-16 char = "#" , hex , hex , hex , hex;
literal special char = "#CR" | "#LF" | "#TAB" | "#SP" | #NULL;
literal hash char = literal utf-8 char | literal utf-16 char | literal special char;
literal char = literal simple char | literal apos char | literal hash char;
```

# strings #
```
literal non-localised string line = "'" , {(char - (eoln | "'")) | "''"} , "'";
literal non-localised heredoc = "<<<" , short identifier , eoln , {char} , eoln , short identifier , eoln;
literal long non-localised string = literal non-localised string line | literal non-localised heredoc;
lnls long = literal long non-localised string , [lnls short];
lnls short = literal hash char , [lnls long | lnls short]; 
literal non-localised string = lnls long | lnls short;
literal string = non-localised string , [">" , short identifier]; 
```

## Example literal strings ##
Here is an example of a heredoc literal string.
```
<<<Banana
Perfection is achieved, not when there is nothing more to add,
but when there is nothing left to take away.
Banana
```

It has two lines. The second line is NOT terminated by a new-line character. The single new-line character after "add," is U+000A . The heredoc introducer identifier (in this example "Banana"), must match case-sensitively to the terminating identifier, and may not be present in the content of the heredoc string.

In this example,
```
'Hello'#SP'world'>Intro
```
the string is 11 characters long in the default language. The string 'Intro' is a key for localisation. If this key is not present, then the string is not localisable.

# guid #
```
uphex = digit | "A".."F";
uphex 4 = uphex | uphex | uphex | uphex;
uphex 8 = uphex 4 | uphex 4;
uphex 12 = uphex 8 | uphex 4;
literal guid = "_guid" , s , uphex 4 , "-", uphex 4 , "-", uphex 4 , "-", uphex 12 , "_";
```
The final _must not be followed by another_.
Example
In pascali, this literal...
```
_guid 3F2504E0-4F89-11D3-9A0C-0305E82C3301_
```
... means this guid...
```
{3F2504E0-4F89-11D3-9A0C-0305E82C3301}
```

# nil #
Nil is a constant.
```
literal nil = "nil";
```

# xml document #
Document is a fundamental type in pascali. You can literal specify a constant document like so...
```
xml document = ?A lexical xml document, either 1.0 fifth edition,
 or 1.1 second edition. XML namespaces 1.1 second edition applies.?;
literal document = "_doc" , s , xml document "_";
```
If an xml decl is present, then
  * standalone must be absent.
  * the encoding pseudo attribute must be absent.
No DTD's are permitted, neither internal nor external. No entities are permitted except character entities and inbuilt xml entities.
Any underscore characters in the document must be escaped as double underscore (`_``_`). The final `_` must not be followed by another `_`.

# xml element #
A literal xml element node, with no parent, may be written as...
```
literal element = "_*" , s , ["version=""1." , ("1"|"0") , """"] , s , xml element , "_";
xml element = ?Lexical xml encoding of an element.?;
```
The element must be stand-alone in terms of namespaces prefixes. The literal element must not be followed by an underscore, and any underscores in the content must be escaped as doubles.

# xml attribute #
```
attri-name = ?xml qname?;
literal attribute = "_@" , s , attri-name , [s] , "=" , [s] , literal string , [s] , "_";
```

# xml text node #
An xml text node is similar to a string. It is different only in that
it is designated as a string, and that it may not be empty.
```
literal xml text node = "_text" , s , literal string , s , "_";
```

# xml namespace #
```
ns prefix = ?Any valid xml namespace prefix.?"
literal namespace = "_ns" , s , "xmlns:" , ns prefix , "=" , literal string , s "_";
```

# TODO #
  * xml comment
  * xml pi
  * array
  * sequence
  * record
  * regex
  * xpath
  * xslt transform
  * xsd schema
  * xpath anonymous function