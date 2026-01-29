# Creating entities

## Table of Content

- [Creating entities](#creating-entities)
  - [Table of Content](#table-of-content)
  - [Creating entities with constraints](#creating-entities-with-constraints)
  - [Equality](#equality)

## Creating entities with constraints

Le but des ComplexProperty est de rajouter des contraintes afin de pouvoir vérifier les données et rajouter des éléments.

```cs
public class Order
{
    public Guid Id { get; set; }

    public Address ShippingAddress { get; set; }
}

public record Address(string Street, string City, string Country);

public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable(TableNames.Orders);

        builder.HasKey(order => order.Id);

        builder.ComplexProperty(i => i.ShippingAddress, address =>
        {
            address.Property(a => a.Street).HasMaxLength(200).IsRequired();
            address.Property(a => a.City).HasMaxLength(100).IsRequired();
            address.Property(a => a.Country).HasMaxLength(100).IsRequired();
        });
    }
}

// On peut map à des Json et ensuite faire des recherches via LinQ
// builder.ComplexProperty(i => i.BillingAddress, address => address.ToJson()); 
```

## Equality

By default, classes are equal if they share the same reference. If you want to add another criteria for equality, your class has to implement the Equatable interface and you have to define the Equals method. You also should implement Equals(object).

If you want you can also override the == and the != operators. You have to override both of them.

If we want to compare values, we will need the IComparable interfaces.
