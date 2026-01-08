# Creating entities

## Table of Content

- [Creating entities](#creating-entities)
  - [Table of Content](#table-of-content)
  - [Creating entities with contraints](#creating-entities-with-contraints)

## Creating entities with contraints

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

// On peut map à des Json et ensuite faire ddes recherches via LinQ
// builder.ComplexProperty(i => i.BillingAddress, address => address.ToJson()); 
```
