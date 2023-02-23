---
title: User Can Update a Patron
category: Vigil Journey
treeid: Vigil/tree/224533ded3cbae7ff65a518e4c0987476275e95f
tags:
- cqrs
- eventsourcing
- factory
- ddd
date: 2016-10-19
---

A simple update of an object is as complex as adding a new instance of the object, which means it is both straight forward and laden with all kinds of caveats and unlying processes. As with [creating a new Patron](/posts/2016/user-can-create-new-patron/), it is trivial for me to put together a contrived example of code that will technically satisfy the requirements for "User Can Update a Patron". A unit Test will create a new Command that will be validated by the Factory and then passed to the MessageQueue for persistance by some other process.

## Command and Tests

There is no surprising code here. The only differences, at this point, between the `CreatePatronCommand` and the `UpdatePatronCommand` is the removal of any fields being required and the addition of an `IKeyIdentity` identifier.

```csharp
using System.ComponentModel.DataAnnotations;
using Vigil.Domain;

namespace Vigil.MessageQueue.Commands
{
    public class UpdatePatronCommand : ICommand
    {
        [Required]
        public IKeyIdentity TargetPatron { get; set; }

        [StringLength(250)]
        public string DisplayName { get; set; }
        public bool? IsAnonymous { get; set; }
        [StringLength(250)]
        public string PatronType { get; set; }
    }
}
```

The intent is that further down the line, the process that actually performs the update will ignore any fields that are _null_. I am partial to the idea of only passing data that actually needs changing. In the future, when I need to handle non-nullable data, then I will worry about how to indicate a removal of information. At this point, though, it is tomorrow-David's problem.

The `IKeyIdentity` is for the issuing code to indicate which entity needs updating. Not all commands will have something like this, as some commands may work on a specific range of entities or all entities.

The unit tests for this command are just as basic as those for the `CreatePatronCommand` class. They are just for making sure that validation can be performed on the object and that it is simple to create a case where it will fail validation.

```csharp
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using Vigil.Domain;
using Xunit;

namespace Vigil.MessageQueue.Commands
{
    public class UpdatePatronCommandTest
    {
        [Fact]
        public void Validation_Requires_TargetPatron()
        {
            UpdatePatronCommand command = new UpdatePatronCommand();

            List<ValidationResult> validationResults = new List<ValidationResult>();
            Validator.TryValidateObject(command, new ValidationContext(command), validationResults, true);

            Assert.Contains(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.TargetPatron)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.DisplayName)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.PatronType)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.IsAnonymous)));
        }

        [Fact]
        public void Validation_On_DisplayName_Has_Maximum_String_Length()
        {
            UpdatePatronCommand command = new UpdatePatronCommand()
            {
                TargetPatron = KeyIdentity.NewIdentity(),
                DisplayName = "This is a string with lots of letters appended.".PadRight(1000, 'A'),
                PatronType = "This is a string with lots of letters appended.".PadRight(1000, 'A'),
            };

            List<ValidationResult> validationResults = new List<ValidationResult>();
            Validator.TryValidateObject(command, new ValidationContext(command), validationResults, true);

            Assert.Contains(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.DisplayName)));
            Assert.Contains(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.PatronType)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.TargetPatron)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(UpdatePatronCommand.IsAnonymous)));
        }
    }
}
```

## Factory and Added Tests

The factory does not need much more than it already has. Adding the `UpdatePatron` method does show that the validation code between the two methods is the exact same, and so I have refactored that into its own protected method. Otherwise, the only real difference is that the `key` for the `QueueCommand` method comes from the `UpdatePatronCommand` instead of being newly generated.

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
            List<ValidationResult> validationResults = ValidateCommand(command);

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

        public FactoryResult UpdatePatron(UpdatePatronCommand command)
        {
            List<ValidationResult> validationResults = ValidateCommand(command);

            if (validationResults.Any())
            {
                return new FactoryResult(validationResults);
            }
            else
            {
                _queue.QueueCommand(command, command.TargetPatron);
                return new FactoryResult(command.TargetPatron);
            }
        }

        protected List<ValidationResult> ValidateCommand(ICommand command)
        {
            List<ValidationResult> validationResults = new List<ValidationResult>();
            Validator.TryValidateObject(command, new ValidationContext(command), validationResults, true);

            return validationResults;
        }
    }
}
```

The added tests are just as easy to write as one would expect. One test to make sure that everything can successfully complete when everything is correct, and one test to make sure that if fails meaningfully.

```csharp
using Moq;
using System;
using Vigil.Domain;
using Vigil.MessageQueue;
using Vigil.MessageQueue.Commands;
using Xunit;

namespace Vigil.Patrons
{
    public class PatronFactoryTest
    {
        // PatronFactory.CreatePatron tests omitted.

        [Fact]
        public void User_Can_Update_a_Patron()
        {
            var queue = new Mock<ICommandQueue>(MockBehavior.Strict);
            queue.Setup(q => q.QueueCommand(It.IsAny<ICommand>(), It.IsAny<IKeyIdentity>())).Verifiable();
            PatronFactory factory = new PatronFactory(queue.Object);
            UpdatePatronCommand command = new UpdatePatronCommand()
            {
                TargetPatron = KeyIdentity.NewIdentity(),
                DisplayName = "Updated Patron Name",
                IsAnonymous = null,
                PatronType = null
            };

            FactoryResult result = factory.UpdatePatron(command);

            queue.VerifyAll();
            Assert.Equal(command.TargetPatron.Id, result.AffectedEntity.Id);
            Assert.Empty(result.ValidationResults);
        }

        [Fact]
        public void User_Cannot_Update_Patron_That_Fails_Validation()
        {
            var queue = new Mock<ICommandQueue>(MockBehavior.Strict);
            queue.Setup(q => q.QueueCommand(It.IsAny<ICommand>(), It.IsAny<IKeyIdentity>())).Verifiable();
            PatronFactory factory = new PatronFactory(queue.Object);

            FactoryResult result = factory.UpdatePatron(new UpdatePatronCommand());

            queue.Verify(q => q.QueueCommand(It.IsAny<ICommand>(), It.IsAny<IKeyIdentity>()), Times.Never);
            Assert.Null(result.AffectedEntity);
            Assert.NotEmpty(result.ValidationResults);
        }
    }
}
```

It feels very anti-climactic to be adding so little code just to "Update" an entity, and it is a little overwhelming to thing of all of the code that needs to be written after this to persist the changes, and all of the code that needs to be written before this to allow a real user to generate the command. However, for this small pocket, the work is complete.