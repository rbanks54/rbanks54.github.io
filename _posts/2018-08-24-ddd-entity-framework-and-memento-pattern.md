---
layout: post
title: "DDD: Entity Framework and the Memento Pattern" 
date: '2018-08-24T12:00:00.001+10:00'
author: Richard Banks
modified_time: '2018-08-24T12:00:00.001+10:00'
---

I worked with a team using Domain Driven Design recently that wanted to use Entity Framework Core (EF Core) for persistence and wanted to ensure that EF concepts didn't leak into their core domain logic.

The first approach was to use mapping code in the infrastructure layer (repositories) to convert between domain entities and EF entities; and yes, the naming overload of EF entities versus domain entities can confuse people at times.

The mapping approach worked well but the code felt a little repetitious and domain entities would often require multiple constructors; one for normal domain logic use and one specifically for use by the mapping code.

Additionally, these secondary constructors don't really belong on entities. They don't represent a domain use case and they have a parameter list that mirrors the entity's properties, at which point people wonder why we don't just make all the properties public and go home.

We wanted something to write code that was cleaner and clearer.

### Enter the memento pattern

With the [memento pattern](https://en.wikipedia.org/wiki/Memento_pattern) an object exposes a representation of its internal state (a memento) and the memento can be used to restore an object to a known state. It's often used for implementing undo operations but nothing stops us using it to support state-based persistence operations.

Let's walk through a simple example using the public transport domain. Remember that in this post a "bus" is a large people moving bus, not a message bus, m'kay? 🙂

We might define a Bus entity in the transport domain as follows:

```cs
public class Bus : IAggregateRoot
{
    public BusId Id { get; }
    public string BusNumber { get; }
    public int SeatedCapacity { get; }
    public int StandingCapacity { get; }

    public Bus(BusId id, string number, int seating, int standing)
    {
        if (string.IsNullOrEmpty(number)) throw new ArgumentOutOfRangeException("number", "bus number must be supplied");
        if (seating <= 0) throw new ArgumentOutOfRangeException("seating");
        if (standing < 0) throw new ArgumentOutOfRangeException("standing");

        Id = id ?? throw new ArgumentNullException("id");
        BusNumber = number;
        SeatedCapacity = seating;
        StandingCapacity = standing;
    }
}
```

There's a few things to note here.

We're representing identity using identity classes (BusId) rather than value types such as `int` or `guid`. It allows our code to match domain concepts more closely and provides a little more type safety when we need one entity to reference another by id.

We're using guard clauses in the constructor to ensure any newly created Bus entity is in a valid state.

### Memento object

So now that we have an entity, we need a class to represent its state.

```cs
public sealed class BusMemento : EntityMemento
{
    public Guid Id { get; private set; }
    public string BusNumber { get; private set; }
    public int SeatedCapacity { get; private set; }
    public int StandingCapacity { get; private set; }

    internal BusMemento(Guid id, string busNumber, int seatedCapacity, int standingCapacity)
    {
        Id = id;
        BusNumber = busNumber;
        SeatedCapacity = seatedCapacity;
        StandingCapacity = standingCapacity;
    }
}

public abstract class EntityMemento
{
    public DateTimeOffset LastModified { get; set; }
}
```

You may have noticed that the memento class is read only. We want changes to state to be made via methods on the Bus entity, not by application code altering a memento object and pushing it back into the domain entity.

We've ignored any guard clauses on the memento constructor as memento instances are only ever created by the domain entity and EntityFramework.

We've also marked the constructor as `internal`. We put our domain logic in a separate assembly to limit the ability for application code to create a memento directly.

If you're wondering, Entity Framework can still create instances of memento objects even when we only have an internal constructor. EF Core 2.1 can now call constructors that have parameters, as long as the parameters names match the properties names. It's worth noting that this mechanism works regardless of the accessibility modifier applied to the constructor. More information can be found on the [official EF Core documentation](https://docs.microsoft.com/en-us/ef/core/modeling/constructors).

> **Side Note:** If you search the internet you'll see a lot of code where the domain entity has a single `State` property containing the memento object, and all methods update that memento object. I would discourage this as it makes using Identity classes and value objects harder and more awkward to use. It also means concepts such as optimistic concurrency (e.g. the `LastModified` property) are now implicitly part of the domain entity, even though they have nothing to do with the entity itself.

### Implementing the memento pattern in the domain entity

To implement the memento pattern within the Bus entity we need two methods.

One to expose a memento to callers, and the other to populate a new instance of a domain entity using a memento.

```cs
public class Bus : IAggregateRoot, IHaveState<BusMemento>
{
    ///...

    public BusMemento State => new BusMemento(Id.Id, BusNumber, SeatedCapacity, StandingCapacity);

    public static Bus FromMemento(BusMemento memento)
    {
        var entity = new Bus(
                new BusId(memento.Id),
                memento.BusNumber,
                memento.SeatedCapacity,
                memento.StandingCapacity
            );
        return entity;
    }
}

public interface IHaveState<S> where S : EntityMemento
{
    S State { get; }
}
```

Nothing too difficult here.

The `FromMemento` approach means we could, optionally, choose to store each and every memento for an entity over time. As the entity definition evolves the structure of a memento may also evolve. Our domain entity could then have multiple FromMemento methods, each knowing how to restore state using any of the different memento formats (versions). While this isn't a common use case, and most people would move to event sourcing if the need arises, it's nice to know that the memento pattern supports this scenario, and that usage of an immutable database such as Datomic is viable with mementos.

### EntityFramework Context

The EntityFramework aspects of our application has only a minor variation to what we might normally have done. We simply ensure that the DBContext works with the memento classes and not the domain entity classes for persistence.

```cs
public class BusContext: DbContext
{
    public DbSet<BusMemento> Buses { get; set; }

    ///...
}
```

### Using this in our application

Now that everything is set we can create a domain entity, ask it for its state, and persist that state in an straightforward manner.

```cs
public static void AddBus(BusContext db, string number, string seated, string standing)
{
    var seatingCapacity = int.Parse(seated);
    var standingCapacity = int.Parse(standing);

    var id = new BusId(Guid.NewGuid());
    var b = new Bus(id, number, seatingCapacity, standingCapacity);

    db.Add(b.State);
    db.SaveChanges();
}
```

Similarly, querying the database to rehydrate domain entities only requires calls to the appropriate `FromMemento()` method. 

```cs
public static IEnumerable<Bus> GetBuses(BusContext db)
{
    var queryResult = from b in db.Buses
                        select Bus.FromMemento(b);

    return queryResult.AsEnumerable();
}
```

### Can we make this even better?

Probably. There's a few things that might make this code easier to work with.

We could split the code into partial classes. One partial containing the domain entity properties and methods, and the holding the memento related code. The trade off is that the memento related code might be forgotten when changes are made. Experiment with it and see what works for you.

You could also nag the C# team and see if they can [get records added to C# 8.0](https://github.com/dotnet/csharplang/issues/39)! Records would mean the memento classes could be represented in a single line of code, rather than a whole lot of boilerplate code. 

### Can I see the sample code?

Of course you can! You'll find a slightly more fleshed out example on GitHub at [https://github.com/rbanks54/ef-and-memento](https://github.com/rbanks54/ef-and-memento).

I hope you find it useful and if you want to talk more about it get in touch via [Twitter](https://twitter.com/rbanks54) or leave an issue on the GitHub repo. Questions and suggestions are always welcome.