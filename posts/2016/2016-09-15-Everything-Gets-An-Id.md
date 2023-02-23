---
title: Everything Gets an Id
category: Vigil Journey
tags:
- vigil
- identifier
date: 2016-09-15
---

There can be a lot of issues with figuring out how to identify an object, especially as I try to track it across various services, platforms, persistance strategies, and even entire systems. For a while, I played around with just having every object contain its own identification field. However, it was painful having to remember to put the same and correct field(s) on every entity. I stumbled upon a blog post (which I cannot find anymore), which talked about pulling the key out into its own class, and have everything inherit from there. I'm not entirely sure I like the idea of a super-base-class, but I think it at least gives me a good place to isolate the key into a globally consistent way.


## Globally Unique Identifier

Having the identifier at the time of initialization turned out to save all kinds of problems for me. When persisting to a database, I no longer have to wait to select the inserted record to find its identity. I can pass entities between services, pass the key to another service, or check on queues by that Guid.

## IKeyIdentity Interface

By abstracting the key to an interface, I can decide later if and how I would like to change it. This will limit the places that are impacted if I need to change out the type of the Id, or add more fields. Also, by having methods use an interface instead of a concrete class, it becomes much easier to mock this when testing. Faking an interface is much easier than building entire test classes.

```csharp
using System;

namespace Vigil.Domain
{
    public interface IKeyIdentity
    {
        Guid Id { get; }
    }
}
```

## KeyIdentity Interface

This is the basic implementation of the `IKeyIdentity` interface. It follows the `ValueObject` convention, making the `Guid` immutable, but also adds the ability to compare any two objects by their `Guid`.

```csharp
using System;
using System.ComponentModel.DataAnnotations;
using System.Diagnostics.Contracts;

namespace Vigil.Domain
{
    public class KeyIdentity : IEquatable<KeyIdentity>, IKeyIdentity
    {
        public static KeyIdentity NewIdentity()
        {
            return new KeyIdentity();
        }

        [Key]
        public Guid Id { get; protected set; }

        protected KeyIdentity()
        {
            Id = Guid.NewGuid();
        }

        protected KeyIdentity(Guid id)
        {
            Id = id;
        }

        public override bool Equals(object obj)
        {
            if (ReferenceEquals(null, obj))
            {
                return false;
            }
            if (ReferenceEquals(this, obj))
            {
                return true;
            }
            if (obj.GetType() != GetType())
            {
                return false;
            }
            return Equals((KeyIdentity)obj);
        }

        public bool Equals(KeyIdentity other)
        {
            if (other == null)
            {
                return false;
            }
            return Equals(other.Id, Id);
        }

        public static bool operator ==(KeyIdentity left, KeyIdentity right)
        {
            return Equals(left, right);
        }
        public static bool operator !=(KeyIdentity left, KeyIdentity right)
        {
            return !Equals(left, right);
        }

        public override int GetHashCode()
        {
            return Id.GetHashCode();
        }

        public override string ToString()
        {
            return Id.ToString();
        }

        [ContractInvariantMethod]
        private void ObjectInvariant()
        {
            Contract.Invariant(Id != Guid.Empty);
        }
    }
}
```