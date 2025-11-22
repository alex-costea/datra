# datra

a language for data transformations.

this is an extremely wip sketch of the language's functionality.

general notes:
- everything is lazy evaluated
- everything is functional. mutation and variables do not exist
- everything is a function and a map, depending on context
- types are regular values

# basic types
- `Undefined := Either!` (terminal object)
- `Nothing := ()!` (initial object)
- `True := 1`
- `False := 0`
- `Truth := Either(True, False)!`

# basic data types
- `Int`
- `Text`
- `Rational`

# type-creating functions
- `Either` - takes positional arguments
- `Type` - takes universe level, 0 by default, higher values need to be explicitly specified

# monads (take one type object, `map` work with them):
- `List`
- `Maybe`

# operations
## basic operators
- `+`, `-`, `*`, `/`, `=`, `=/=`, `>`, `>=`, `<`, `<=` with their usual senses
- `+` also for string concatenation
- `Int` \ `Int` is `Int`. if one of the operators is Float, it is Float
## `!` evaluates a function
- applying a map to a function does not ever automatically evaluate a function
- note that this doesn't necessarily break the laziness. `a := f b!` will still wait for `a` itself to be evaluated
## `()` creates a map
- maps have keys and values: `(identifier1 := 1, identifier2 := 2)`
- they maintain their ordering, but it doesn't have semantic effect
- keys can be identifiers, `String` (semantically equivalent to identifiers) or any other value
- if identifier is skipped, it is given as the smallest `Int` larger than 0 which isn't already taken
- maps can be partial: `(x : Int)`. when an argument is partial, a type definition must be provided.
- the argument type definition is automatically inferred if given
- maps can reference their own identifiers: `(x := 1, y := x)`
- `do` syntax: `do op1; op2;...; return map_obj` where all operations are `name := value`, which will not be included in `map_obj`
- a map subtypes another map if all the arguments of the supertype have a match in the subtype, and their types are covariant
- all values are maps, specifically `x` is equivalent to `(x)`, which itself is equivalent to `(0 : x)`
## `->` creates a function
- every map subtypes the function from the initial object to itself
- syntax: `map1 -> map2`
- example: `(x : Int, y : Int) -> x + y`
- note that `Int` objects (and all non-map objects) fit the map type by automatic conversion
- every function subtypes map (from all possible first map values to resulting second map values)
- as it is simply `map1 -> map2`, function arguments can have default values. example: `(x := 5) -> x`
- identifier names from the origin map can be used automatically
- non-identifier arguments can be accessed using the `arg` keyword: `(0 : Int) -> 2 * arg 0`
- function subtyping is contravariant on input map and covariant on output map
## `f m` applies a map to a function
- if all its map arguments have been provided, it can be evaluated with `!`
- you can partially apply a function, in which case it won't be evaluable until all arguments are provided
- example: `sum := (x : Int, y : Int) -> x + y; four_adder := sum (x := 4); result := four_adder(y := 5)!`
# `>>` pipes data to function
- `func data` is equivalent to `data >> func`
- `>>` is right associative. `data >> data >> func` is `data >> (data >> func)`

# hello world

```
main := "hello, world!"
```

the function called `main` is automatically applied at runtime

# potential fizzbuzz (functionality not sufficiently defined yet)

```
main := (n : Int) -> do
    list := 1...n+1
    mapping := map(
        (x -> x % 15 == 0, "FizzBuzz") >>
        (x -> x % 5 == 0, "Buzz") >>
        (x -> x % 3 == 0, "Fizz") >>
        switch
    )
    return mapping list!
```

note that n will have to be piped.
