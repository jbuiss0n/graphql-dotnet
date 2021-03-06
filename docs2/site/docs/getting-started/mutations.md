# Mutations

To perform a mutation you need to have a root Mutation object that is an `ObjectGraphType`.  Mutations make modifications to data and return a result.  You can only have a single root Mutation object.  Mutations are executed serially.

> See the [official GraphQL documentation on mutations](http://graphql.org/learn/queries/#mutations).

Instead of using the `query` keyword, you are required to use `mutation`.  Similar to a `query`, you can omit the `Operation` name if there is only a single operation in the request.

```graphql
mutation ($human:HumanInput!) {
  createHuman(human: $human) {
    id
    name
  }
}
```

The JSON request for this mutation would look like:

```json
{
  "query": "mutation ($human:HumanInput!){ createHuman(human: $human) { id name } }",
  "variables": {
    "human": {
      "name": "Boba Fett",
      "homePlanet": "Kamino"
    }
  }
}
```

Set the `Mutation` property on your `Schema`.

```csharp
public class StarWarsSchema : Schema
{
  public StarWarsSchema(IDependencyResolver resolver)
    : base(resolver)
  {
    Query = resolver.Resolve<StarWarsQuery>();
    Mutation = resolver.Resolve<StarWarsMutation>();
  }
}
```

A `mutation` `GraphType` looks idential to a `query` `GraphType`.  The difference is you are allowed to mutate data.

```csharp
public class StarWarsMutation : ObjectGraphType
{
  public StarWarsMutation(StarWarsData data)
  {
    Field<HumanType>(
      "createHuman",
      arguments: new QueryArguments(
        new QueryArgument<NonNullGraphType<HumanInputType>> {Name = "human"}
      ),
      resolve: context =>
      {
        var human = context.GetArgument<Human>("human");
        return data.AddHuman(human);
      });
  }
}
```

To provide a set of input values you must use `InputObjectGraphType`.

```csharp
public class HumanInputType : InputObjectGraphType
{
  public HumanInputType()
  {
    Name = "HumanInput";
    Field<NonNullGraphType<StringGraphType>>("name");
    Field<StringGraphType>("homePlanet");
  }
}
```

`StarWarsData` is an in-memory data store.

```csharp
public class StarWarsData
{
  private List<Human> _humans = new List<Human>();

  public Human AddHuman(Human human)
  {
    human.Id = Guid.NewGuid().ToString();
    _humans.Add(human);
    return human;
  }
}
```

> See the [StarWars example](https://github.com/graphql-dotnet/examples/tree/master/src/StarWars) for a full implementation.
