---
title: A Word On Validation
category: Vigil Journey
tags:
- cqrs
- factory
- ddd
- validation
date: 2016-10-07
---

For a while, I have been battling with where validation should occur, and who is responsible for ensuring that the information in a command is good data. For the first layer of data validation, I am turning to the Command to validate itself — the mere application of data types is a rudimentary form of data validation, so including addition simple validation extends that basic logic.

## Simple Validation

Taking a queue from the [`IValidatableObject`](https://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.ivalidatableobject.aspx) interface, simple validation logic will be handled by the command itself. I am defining _simple_ as validation that requires no external knowledge.

```csharp
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using Vigil.Domain;

namespace Vigil.MessageQueue.Commands
{
    public class CreatePatronCommand : ICommand
    {
        [Required, StringLength(250)]
        public string DisplayName { get; set; }
        [DefaultValue(false)]
        public bool IsAnonymous { get; set; } = false;
        [Required, StringLength(250)]
        public string PatronType { get; set; }
    }
}
```

This validates that both the `DisplayName` and the `PatronType` are not longer than 250 characters and are not empty. There is no validation as to whether the `PatronType` exists in some look-up table, or whether the `DisplayName` is unique (or if it even needs to be unique).

## Why A Command Does Not Require Itself Be Valid

Even though the Command knows _how_ to validate itself, I have decided that the command is not required to actually ensure that it is valid before work is performed on it — for two reasons. First, a Command is merely a carrier for information, and only understands simple rules governing whether the data is in a valid state. There are many cases where it is acceptable for a Command to be in an invalid state — when logging failed attempts, or when saving an in progress state. Second, a Command does not actually know when work is being performed on its information. Other objects (the message queue, the command handler, the command logger) are responsible for deciding if the data needs to be fully valid and then whether the data actually is valid. This was a bit of a struggle for me early on, because if the object knows _how_ to validate itself, shouldn't it also make sure that it is always in a valid state? It was when I started thinking about persisting partial states and persisting failed states that I began to realize how to solve this conundrum.

## Who Validates the Validation?

In my current system design, I am relying on the Factory to handle validating that a Command's data has passed all of its internal data integrity checks. As I've already mentioned the `IValidatableObject`, it should come as no surprise that I intend to use the `DataAnnotations` namespace to perform the validation.

```csharp
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using Vigil.Domain;
using Vigil.MessageQueue;
using Vigil.MessageQueue.Commands;

namespace Vigil.Patrons
{
    public class PatronFactory
    {
        protected readonly ICommandQueue _queue;

        public PatronFactory(ICommandQueue queue)
        {
            _queue = queue;
        }

        public FactoryResult CreatePatron(CreatePatronCommand command)
        {
            List<ValidationResult> validationResults = new List<ValidationResult>();
            Validator.TryValidateObject(command, new ValidationContext(command), validationResults, true);

            if (validationResults.Any())
            {
                return new FactoryResult(validationResults);
            }
            else
            {
                var key = KeyIdentity.NewIdentity();
                _queue.QueueCommand(command, key);
                return new FactoryResult(key);
            }
        }
    }
}
```

There are some odd quirks that I will have to remember, the primary (at this point) being the order that validation occurs, and when it aborts further validation. Jeff Handley goes into more detail in his post [Validating Objects and Properties with Validator](http://jeffhandley.com/archive/2009/10/16/validator.aspx), but the key piece that I took away from it that I hadn't figured out while playing with the API is this:

> 1. Validate property-level attributes
> 1. If any validators are invalid, abort validation returning the failure(s)
> 1. Validate the object-level attributes
> 1. If any validators are invalid, abort validation returning the failure(s)
> 1. Call its Validate method and return any failure(s)

## Complex Validation

I have not yet solved the problem of how to validate a command that requires external knowledge. At some point, I am going to need to be sure that a command has valid information with regards to all other information in a system. Account numbers must be unique and sequential; transaction dates must be the same as the date of the batch; schedule effective date ranges cannot overlap. Whatever does the validation will have to have knowledge of the state of other common data points - maybe it doesn't specifically know what the persistence technology is, but the validating object will need to know how to compare the Command's data with persisted data. Who gets this priviledge? I have not yet figured that one out.