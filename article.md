![Ruby](images/Ruby_logo.svg)
# Pattern Matching


Was *experimental* in Ruby **2.7**. But is stable since **3.0**.
Some feature where also added and stable in later versions.


*from Ruby doc:*

Pattern matching is a feature allowing deep matching of structured values by
<span class="fragment highlight-blue" data-fragment-index="1">checking</span>
the structure and
<span class="fragment highlight-blue" data-fragment-index="2">binding</span>
the matched parts to local variables.

In other words: data validation and extraction. <!-- .element: class="fragment" data-fragment-index="3" -->


⚠️ It has nothing to do with **String** nor **Regexp** but with data strutures,
like **Array** and **Hash** (but not limited to)



# Patterns and expressions


They are 4 different patterns you can use
- Value <!-- .element: class="fragment fade-up" -->
- Array <!-- .element: class="fragment fade-up" -->
- Hash <!-- .element: class="fragment fade-up" -->
- Find (Stable in 3.2) <!-- .element: class="fragment fade-up" -->

Note:
Any pattern can be nested inside array/find/hash pattern <!-- .element: class="fragment fade-in" -->


And two kind of expressions


`case`/`in`

```ruby
case <expression>
in <pattern1>
  ...
in <pattern2>
  ...
in <pattern3>
  ...
else
  ...
end
```

Note:
Note that `in` and `when` branches can **NOT** be mixed in one `case` expression


standalone expression with `=>` or `in` operators

```ruby
<expression> => <pattern>

<expression> in <pattern>
```


👮‍♀️

`case`/`in` and `=>` expression are **exhaustive**.

`NoMatchingPatternError` is raised if no match

Note:
Unless there is an `else` branch for `case`/`in`.



# checking 🔍


### Value pattern
```ruby
case 1
in Integer
  "it's an int"
in String
  "it's a string"
else
  "something else"
end
#=> "it's an int"
```
It behaves like `case`/`when` and use the `===` operator

It is meant to be mixed with other patterns


### Reminder
```ruby
Integer === 1 #=> true
```
```ruby
1 === Integer #=> false
```
```ruby
1 === 1 #=> true
```
```ruby
"hello there!" === "hello world!" #=> false, like `==`
```
```ruby
(1..10) === 5 #=> true
```
```ruby
/^Hello/ ===  "Hello there!" #=> true
```


### Array pattern
```ruby
case [1, 2]
in [Integer, Integer]
  "matched"
else
  "not matched"
end
#=> "matched
```
pattern matches the whole Array


### Hash pattern
```ruby
case { a: 1, b: 2, c: 3 }
in { a: Integer }
  "matched"
else
  "not matched"
end
#=> "matched"
```
matches even if there are other keys

(except for `{}`)


```ruby
case { name: "Riri", age: 14 }
in { name: String, age: 18.. }
  "and adult"
else
  "not an adult"
end
#=> "not an adult"
```


Syntaxic sugar FTW 🍭

parenthesis can be omitted

```ruby
case [1, 2]
 in Integer, Integer
   "matched"
 else
   "not matched"
 end

 case { a: 1, b: 2, c: 3 }
 in a: Integer
   "matched"
 else
   "not matched"
 end
```


### Find pattern
similar to array pattern but it can be used to check if the given object has any elements that match the pattern

experimental since 3.0 ; stable in 3.2


```ruby
contacts = [
  { name: "Riri" },
  { name: "Fifi" },
  { name: "Feel Good Inc.", company: true },
  { name: "Loulou" }
]
case contacts
in [*, { company: true }, *]
  "at least one company"
else
  "no company at all"
end
#=> "at least one company"
```


### Rest
Both array and hash patterns support *rest* operator


```ruby
case [1, 2, 3]
in [Integer, *]
  "matched"
else
  "not matched"
end
#=> "matched"
```

```ruby
case { a: 1, b: 2, c:  3 }
in { a: Integer, ** } # same with or without
  "matched"
else
  "not matched"
end
#=> "matched"
```


if there should be no other keys
```ruby
case { a: 1, b: 2, c:  3 }
in { a: 1, **nil }
  "matched"
else
  "not matched"
end
#=> not matched
```


### Alternative pattern
```ruby
case { a: 1, b: 2, c: 3 }
in { a: 1, **nil } | { b: 2, c: 3 }
  "matched"
else
  "not matched"
end
#=> matched
```

Note:
Simply said: combining patterns



# binding 🔗

Note:
Besides checks, a very important features of pattern matching is
binding of matched parts to local variables.


Basic form

`<pattern> => <local variable>`
```ruby
case [1, 2]
in Integer => int, Integer
  "matched: #{int}"
else
  "not matched"
end
#=> "matched: 1"
```
```ruby
case { a: 1, b: 2, c: 3 }
in a: Integer => int
  "matched: #{int}"
else
  "not matched"
end
#=> "matched: 1"
```

Note:
The basic form of binding is just specifying => variable_name after the matched (sub)pattern (one might find this similar to storing exceptions in local variables in a rescue ExceptionClass => var clause):


More syntaxic sugar ! 🍬
```ruby
case [1, 2]
in int, Integer
  "matched: #{int}"
else
  "not matched"
end
#=> "matched: 1"
```
```ruby
case { a: 1, b: 2, c: 3 }
in a: int
  "matched: #{int}"
else
  "not matched"
end
#=> "matched: 1"
```

Note:
⚠️ It does not work with *alternative pattern*


```ruby
case [1, 2, 3]
in int, *rest
  "matched: #{int}, rest: #{rest}"
else
  "not matched"
end
#=> "matched: 1, rest: [2, 3]"
```
```ruby
case { a: 1, b: 2, c: 3 }
in a: int, **rest
  "matched: #{int}, rest: #{rest}"
else
  "not matched"
end
#=> "matched: 1, rest: { :b => 2, :c => 3 }"
```


# => / in operators

Note:
Same behavior but much shorter syntax for simpler use case


```ruby
<expression> in <pattern>
# true or false
```

```ruby
<expression> => <pattern>
# nil or raise NoMatchingPatternError
```


`in` as condition
```ruby
if current_user in { admin: true }
  "user is admin"
else
  "lambda user"
end
```


`=>` as guard clause
```ruby
def do_action
  current_user => { admin: true }

rescue NoMatchingPatternError
  redirect_to root_path, alert: "nope!"
end
```

Note:
I also see as a way to explicitly write expected "hash schema",
and avoiding unclear `undefined method [] for NilClass`


Checking and extracting (inspired from real example)
```ruby
def extract_full_name_parts(extracted_line)
  extracted_line => { full_name: String => full_name }
  full_name.match(regexp) => { last_name:, first_name: }

  # stuff with `last_name` and `full_name` local variables
rescue NoMatchingPatternError
  extracted_line.error!("full_name is missing or incomplete")
end
```



## variable pinning 📌

Note:
Due to the variable binding feature, existing local variable can not be straightforwardly used as a sub-pattern:


```ruby
expectation = 18

case [1, 2]
in expectation, *rest
  "matched. expectation was: #{expectation}"
else
  "not matched. expectation was: #{expectation}"
end
# expected: "not matched. expectation was: 18"
# real: "matched. expectation was: 1" -- local variable just rewritten
```
```ruby
expectation = 18
case [1, 2]
in ^expectation, *rest
  "matched. expectation was: #{expectation}"
else
  "not matched. expectation was: #{expectation}"
end
#=> "not matched. expectation was: 18"
```



# advanced/real life examples 🧠


```ruby
case find_registered_contact(contact_attributes)
in Contact => contact
  { _other_attrs:, contact_id: contact.id }
in ContactToCreate[String => tmp_contact_id, _line]
  # contact does not exist yet and needs to be created
  { _other_attrs:, tmp_contact_id: }
end
```


pinning extracted variable within pattern
```ruby
case { riri: "Duck", fifi: "Duck", loulou: "Duck" }
in { riri: last_name, fifi: ^last_name, loulou: ^last_name }
  "They are all brothers, from #{last_name} family"
else
  "They are not from the same family"
end
#=> "They are all brothers, from Duck family"
```


if/unless 🤯
```ruby
extract_company_name = false

contacts_data = [
  { name: "Feel Good Inc.", company: true },
  { name: "Riri", last_name: "Duck" },
  { name: "Fifi", last_name: "Duck" },
  { name: "Loulou", last_name: "Duck" },
  { name: "Charlie" }
]
```
```ruby
case contacts_data
in [*, { company: true, name: }, *] if extract_company_name
  "company: #{name}"
in [*, { name:, **nil }, *] if name == "Charlie"
  "I found Charlie!"
in [*, { company: true }, *]
  "at least one company present"
else
  "no company found"
end
#=> "I found Charlie!"
```



### Pattern matching on other kind of objects


### use deconstruct / deconstruct_keys
```ruby
class Point
  def initialize(x, y)
    @x, @y = x, y
  end

  def deconstruct
    puts "deconstruct called"
    [@x, @y]
  end

  def deconstruct_keys(keys)
    puts "deconstruct_keys called with #{keys.inspect}"
    { x: @x, y: @y }
  end
end
```


```ruby
case Point.new(1, -2)
in px, Integer  # sub-patterns and variable binding works
  "matched: #{px}"
else
  "not matched"
end
# deconstruct called
#=> "matched: 1"
```


```ruby
case Point.new(1, -2)
in x: 0.. => px
  "matched: #{px}"
else
  "not matched"
end
# deconstruct_keys called with [:x]
#=> "matched: 1"
```


### deconstruct implementations

- [Time](https://docs.ruby-lang.org/en/master/Time.html#method-i-deconstruct_keys)
- [MatchData](https://docs.ruby-lang.org/en/master/MatchData.html#method-i-deconstruct)
- [Data](https://docs.ruby-lang.org/en/master/Data.html#method-i-deconstruct)



# More
[Pattern Matching](https://docs.ruby-lang.org/en/master/syntax/pattern_matching_rdoc.html)
[pattern syntax](https://docs.ruby-lang.org/en/master/syntax/pattern_matching_rdoc.html#label-Appendix+A.+Pattern+syntax)
