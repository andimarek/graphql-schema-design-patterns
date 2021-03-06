
# GraphQL schema design patterns

A list of GraphQL schema design patterns.

1. [Argument as query field](#argument-as-query-field) 
1. [Argument as formatter](#argument-as-formatter)
1. [Different fields for the semantically same value](#different-fields-for-the-semantically-same-value)
1. [Interface with multiple implementations](#interface-with-multiple-implementations)
1. [Explicit type field](#explicit-type-field)
1. [Generic object with type field](#generic-object-with-type-field)
1. [Union with weak interface](#union-with-weak-interface)
1. [Any Object (or JSON)](#any-object-or-json)
1. [Relay pagination](#relay-pagination)
1. [Wrap scalar in object](#wrap-scalar-in-object)
1. [Error types](#error-types)
1. [Top level groups](#top-level-groups)
1. [Specific return type for each Mutation](#specific-return-type-for-each-mutation)
1. [One specific input type for each Mutation](#one-specific-input-type-for-each-mutation)

## Argument as query field

The argument of a field serves as a query parameter to select the field.

### Example
The id of an user is used to select the user object or a search for multiple users based on the name.

``` graphql
type Query {
  user(id: ID): User
  userSearch(name: String): [User]
}
type User {
  id: ID
  ...
}
```
Or based on 

### Discussion

The argument acts as a query parameter to select one (or more) values.

## Argument as formatter 

The argument of an field serves as a formatter.

### Example
Instead of returning a fixed date format the format argument decides on the returned format.

```graphql
type User {
  dateOfBirth(format: String): String
}
```

It also possible to use this pattern for non Scalars:

```graphql
type WebPage {
  content(contentFormat: WebPageFormat): RenderedWebPage   
}
enum WebPageFormat {
  HTML,
  TEXT
}
interface RenderedWebPage {
  value:String
}
type Html implements RenderedWebPage {
...
}
type Text implements RenderedWebPage {
...
}
```
### Discussion
The difference to [Argument as query field](#argument-as-query-field) is that the argument doesn't really specify what you want, but rather how you want it. The `content` and the `dateOfBirth` are already selected implicitly by the `User` or `WebPage`, the question is only the format.

## Different fields for the semantically same value

Using different fields for the (semantically) same value.

### Example
```graphql
type User {
  dateOfBirthUTC: String
  dateOfBirthLocale: String
}

```
### Discussion

This is the alternative to [Argument as formatter](#argument-as-formatter) where you can select the format by providing an argument. If both formats are needed as the same time this pattern allows for it directly:
``` graphql
...on User {
  dateOfBirthUTC,
  dateOfBirthLocal
}
```
while [Argument as formatter](#argument-as-formatter) requires the usage of aliases.


## Interface with multiple implementations

Multiple objects implementing a interface and sharing some fields.

### Example
``` graphql
interface Item {
  title: String
}
type Book implements Item{
  title: String
  pageCount: Int
}

type Movie implements Item{
  title: String
  director: String
}

```

## Explicit type field
Having an explicit type field instead of relying on `__typename`.

### Example
``` graphql
interface Item {
  title: String
  # static value for each Item type
  type: String
}

type Book implements Item {
  title: String
  type: String
  pageCount: Int
}

type Movie implements Item {
  title: String
  type: String
  director: String
}

```

## Discussion
Every type name can be queried with the introspection field `__typename`. This is automatically the name of the current object. If for some reason this automatic behavior is not suitable a explicit type field can be added. 


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
  # only available for Books
  pageCount: Int
}
enum ItemType {
  BOOK,
  MOVIE
}

```

### Discussion
One difference to [Interface with multiple implementations](#interface-with-multiple-implementations) is the way it can be queried:

```graphql
...on Item {
  title
  director
  pageCount
}
```
This allows for simpler queries without fragments for both types.


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

## Wrap scalar in object

Using a object type to wrap a single scalar value.

### Example
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
### Discussion
This allows for further non-breaking changes of the schema on the costs of having more complex queries in the beginning.
This is also an alternative to have custom scalars: Instead of having a Country scalar you have Country type. But scalars can be used as input and output types. This means a second `input CountryInput ...` input type is maybe needed.

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

Especially useful for larger APIs.
The same pattern is not possible for Mutations, because only the top level mutation fields are guaranteed to be executed sequentially.

## Specific return type for each Mutation

Each mutation field returns a specific type, just for this Mutation. 

### Example
```graphql
type Mutation {
  addUser(input: AddUserInput): AddUserInputPayload
}
type AddUserInputPayload {
  user: User
  clientMutationId: String
}
```

## One specific input type for each Mutation

Mutation becomes one single input argument. 

### Example
```graphql
type Mutation {
  addUser(input: AddUserInput): AddUserInputPayload
}
input AddUserInput {
  name: String
  ...
}

```
### Discussion
Related to [Specific return type for each Mutation](#specific-return-type-for-each-mutation), but for the input type.
