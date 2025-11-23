# datra

a language for data transformations.

this is an *extremely* wip sketch of the language's functionality. do not take anything as a final word.

i also have to make sure all of this is actually logically sound.

general notes:
- everything is lazy evaluated
- everything is functional. mutation and variables do not exist
- everything is a function and a map, depending on context
- types are regular values (higher universe values need to be explicitly specified)

# basic values
- `Undefined := []` (terminal object)
- `Nothing := ()` (initial object)
- `Anything` (universal supertype)
- `Truth := [True := Nothing | False := Nothing]`

# basic data types
- `Int`
- `Text`
- `Rational`

# operations
## basic operators
- `+`, `-`, `*`, `/`, `=`, `=/=`, `>`, `>=`, `<`, `<=` with their usual senses
- `and`, `or, `not` likewise
- `+` also for text concatenation
- `Int` \ `Int` is `Int`. if one of the operators is Float, it is Float
## `!` evaluates a function
- applying a map to a function does not ever automatically evaluate a function
- note that this doesn't necessarily break the laziness. `a := f b!` will still wait for `a` itself to be evaluated
## `()` creates a map
- maps have keys and values: `(identifier1 := 1, identifier2 := 2)`
- they maintain their ordering, but it doesn't have semantic effect
- keys can be identifiers, `Text` (semantically equivalent to identifiers) or any other value
- if identifier is skipped, it is given as the smallest `Int` larger than 0 which isn't already taken
- maps can be partial: `(x : Int)`. when an argument is partial, a type definition must be provided.
- the argument type definition is automatically inferred if a value given
- `do` syntax: `do op1; op2;...; return map_obj` where all operations are `name := value`, which will not be included in `map_obj`
- a map subtypes another map if all the arguments of the supertype have a match in the subtype, and their types are covariant
- all values are maps, specifically `x` is equivalent to `(x)`, which itself is equivalent to `(0 : x)` and `() -> (0 : x)`
- map keys can be an entire type, which all gets mapped to one value: `(*Text := "Text")`
- more generally, when applied with an argument, maps check the first key which supertypes the value
## `[]` creates a sum type
- delineate possible values using `|`
- same general rules apply as for maps, sum types have a key and value, and if a key isn't given, it will use the correct Int value
- note that if a map has any sum types for any of its values, it cannot be considered filled
## `->` creates a function
- syntax: `map1 -> map2`
- example: `(x : Int, y : Int) -> x + y`
- note that `Int` objects (and all non-map objects) fit the map type by automatic conversion
- every function subtypes some potentially infinite map (from all possible first map values to all possible second map values)
- every map subtypes some function, which can be obtained by creating a sum type on both input/output from the map's product structure
- as it is simply `map1 -> map2`, function arguments can have default values. example: `(x := 5) -> x`
- identifier names from the origin map can be used automatically
- all arguments in a function body are within the map `arg`, which can also be treated as a function: `*Int -> 2 * arg(0)!`
- function subtyping is contravariant on input map and covariant on output map
## `f m` applies a map to a function
- if all its map arguments have been provided, it can be evaluated with `!`
- you can partially apply a function, in which case it won't be evaluable until all arguments are provided
- example: `sum := (x : Int, y : Int) -> x + y; four_adder := sum (x := 4); result := four_adder(y := 5)!`
## `><` concatenates two maps
- if there is overlap, values from the second map override values from the first map
## `*type` is sugar for `(0 : type)`

# emergent features
- when a function  `*[Int | Text] -> *Truth` is defined using a map, you get pattern matching:

```
my_func : *[Int | Text] -> *Truth := (
    *Int := True,
    *Text := False
)
```
- lazy evaluation allows recursive type definitions, e.g. `Tree := [(left : Tree, right: Tree) | Nothing]`
- maps can reference their own identifiers: `(x := 1, y := x)`
- since functions are maps, `*Int -> 2 * arg(0)!` is also the map `(0: 0, 1 : 2, 2 : 4, ...)`
- `(x : Text) >< (Int -> [Int | Nothing]) -> Nothing` can take an arbitrary number of positional args
- similarly `(x : Text) >< (Text -> [Int | Nothing]) -> Nothing` can take any identifier - value pairs