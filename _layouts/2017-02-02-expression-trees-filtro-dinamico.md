---
title: Expression Trees Lambda Expressions C#
updated: 2017-02-02 08:56
---

Las <a href='https://msdn.microsoft.com/es-es/library/bb397687.aspx' title='Msdn: guía de uso expresiones lambda'>expresiones Lambda</a> ya llevan un tiempo en C#, no es nada nuevo. Las usamos habitualmente cuando filtramos colecciones con Linq pero por norma general no necesitamos componer nosotros un árbol de expresiones. Hoy veremos un ejemplo donde podría aplicar el uso los Expression Trees que nos proporcionan las Lambda Expressions de C#. 
<br>  
``` java
Task<IEnumerable<MyEntity>> GetByTypeAndCaducity(DateTime caducity, MyEntityType[] types)
{
    var entitiesNotDeprecated = MyEntities
        .Where(x => x.EndValidityDate.Day == caducity.Day);


    ParameterExpression pe = Expression.Parameter(typeof(MyEntity), "MyEntity");
    Expression predicate = Expression.Constant(false);

    foreach (var type in types)
    {
        Expression left = Expression.Property(pe, "MyType");
        Expression right = Expression.Constant(type);
        Expression e = Expression.Equal(left, right);
        predicate = Expression.OrElse(predicate, e);
    }


    MethodCallExpression whereCallExpression = Expression.Call(
        typeof(Queryable),
        "Where",
        new Type[] { entitiesNotDeprecated.ElementType },
        entitiesNotDeprecated.Expression,
        Expression.Lambda<Func<TillCoupon, bool>>(predicate, new ParameterExpression[] { pe }));


    return await entitiesNotDeprecated.ToListAsync();
}
```









