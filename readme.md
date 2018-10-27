# GraphQL schema design patterns

## Argument as query field

The argument of a field serves as a query parameter to select the field:

### Example
The id of an user is used to select the user object

``` graphql
type Query {
  user(id: ID): User
}
type User {
  id: ID
  ...
}
```

## Argument as formatter for a Scalar

The argument of an field serves as a formatter for a Scalar:

### Example
Instead of returning a fixed date format the format argument decides on the returned format.

```graphql
type User {
  dob(format: String): String
}
```

## Interface with multiple implementations

Multiple objects implementing a interface and sharing some fields.

## Generic object with type field

A generic objects which actually represents multiple types. It has a type field to recognize the actual type.
Not every field makes sense for every actual type.

### Example
A generic Item object which can be a book or a movie.

```graphql
type Item {
  type: ItemType
  title: String
  # only available for Movies
  director:String
}
enum ItemType{
  BOOK,
  MOVIE
}

```

## Union with weak interface
A weak interfaces implemented by a lot of objects and a union type restricting the possible implementations.

### Example

```graphql
type Node {
  id: ID
}

type Human implements Node {
  id: ID
  friends: [HumanFriend]
  ...
}

union HumanFriend = Dog | Human

type Dog implements Node {
  id: ID
  ...
}  

type House implements Node {
  id:ID
  ...
}

```
### Discussion

It is useful to restrict the possible return types if the common interface is to broad. In the above example `Node` is to generic and a Human and Dog don't have enough in common to extract another interface. 

## Any Object (or JSON)

Opting out of the GraphQL type system by returning or accepting a general Object type (or JSON type as often described).

## Relay pagination

Using the relay.js way of doing pagination: https://facebook.github.io/relay/graphql/connections.htm

## Object instead of scalar 

Using a object type instead of scalar to be more future proof.

### Example:
Using a complex country type instead of a string.

``` graphql
type Address{
  country: Country 
}

type Country {
  code: String
  # possible more fields in the future
}
```

## Error types

Instead of using the normal GraphQL error handling, define explicit Error types.

## Top level groups  

Having top level fields without any meaning except grouping the underlying fields.

### Example
```graphql
type Query {
  humans: Humans
  animals: Animals
}
type Humans {
  human(id:ID): Human
  food: [HumanFood]
}

type Animals {
  animal(id: ID): Animal
  food: [AnimalFood]
} 
...
```
### Discussion


