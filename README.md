# Customm filter in [Hot Chocolate](https://chillicream.com/docs/hotchocolate)

Hey, if you want to add case insensitive custom filter operation in hotchocolate then you can find solution below.

For ex : 
```
query {
  users(where: { username: { contains: "admin" } }) {
    firstname
    lastname
  }
}
```

Solution :
```
protected override void Configure(IFilterConventionDescriptor descriptor)
{
    descriptor.AddDefaults();
    descriptor.Provider(
        new QueryableFilterProvider(
            x => x
                .AddDefaultFieldHandlers()
                .AddFieldHandler<QueryableStringInvariantContainsHandler>()));
}
```

```
public class QueryableStringInvariantContainsHandler : QueryableStringOperationHandler
{
    private static readonly MethodInfo _contains = typeof(string).GetMethod("IndexOf",
            new[] { typeof(string), typeof(StringComparison) });

    protected override int Operation => DefaultFilterOperations.Contains;
    public override Expression HandleOperation(QueryableFilterContext context, IFilterOperationField field, IValueNode value, object parsedValue)
    {
        Expression property = context.GetInstance();
        if (parsedValue is string str)
        {
            return Expression.NotEqual(
                    Expression.Call(
                        property,
                        _contains,
                        Expression.Constant(str, typeof(string)),
                        Expression.Constant(StringComparison.InvariantCultureIgnoreCase, typeof(StringComparison))
                    ),
                    Expression.Constant(-1, typeof(int))
                );
        }
        throw new InvalidOperationException();
    }
}
```

Add filter convention in Startup,cs
```
services.AddGraphQLServer()
                    .AddFiltering<CustomFilteringConvention>()
                    .AddConvention<IFilterConvention>(
                        new FilterConventionExtension(
                            x => x.AddProviderExtension(
                                new QueryableFilterProviderExtension(
                                    y => y.AddFieldHandler<QueryableStringInvariantContainsHandler>()))));
```

you can also refer this links too : 
1. [C# Dynamic lambda problem with implementing "IndexOf" ignore case](https://dynamic-linq.net/knowledge-base/57830194/csharp-dynamic-lambda-problem-with-implementing--indexof--ignore-case)
2. [Extending Filtering](https://chillicream.com/docs/hotchocolate/api-reference/extending-filtering#extending-iqueryable)
