---
title: User Cannot* Create Patron That Fails Validation**
category: Vigil Journey
tags:
- vigil
- cqrs
- factory
- ddd
- validation
date: 2016-09-18
---

Creating a patron (by which I mean, issuing a command such that some process somewhere will instantiate an actual Patron entity, and perhaps persist it in some useful manor) is all well and good, until someone tries to stuff in data that isn't valid. Users like to do things like this all the time, so I am going to attempt to protect against it from the beginning. Additionally, by setting up the mechanisms to test validation and perform validation, I should be able to keep a culture of enforcing these conventions throughout development.


## *"Cannot"

With everything revolving around commands, telling a user something can't be done is less accurate, and it is more about declining to queue the command and letting the user know why. This starts by actually validating the command in question.

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

## **"Validation"

Who actually performs the validation is questionable. For now, I have decided to place the onus on the command to know what makes it valid. As a first step, I am just using the `DataAnnotations` attributes, as they serve my needs for now. In the future, I know that I will need to do look-up validation (is the `PatronType` valid), but for now I am just demonstrating that the bare minimum works.

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

## Testing the Code

```
CreatePatronCommandTest:
  ✅ Validation_On_DisplayName_Has_Maximum_String_Length
  ✅ Validation_Requires_DisplayName_and_PatronType

PatronFactoryText
  ✅ User_Can_Create_New_Patron
  ✅ User_Cannot_Create_Patron_That_Fails_Validation
```

Of course, since new code has been added, new tests need to be made. Following the Microsoft convention for where tests are located, the solution is broken into two root folders: src and test. There is a one-to-one matching of projects with real code in the src folder, and projects that hold the unit tests for those projects. Test projects have the same name as their target, suffixed with '.Tests' (eg. `src\Vigil.Domain` and `test\Vigil.Domain.Tests`). The folder structure in the two projects should be identical, and all test classes have the same name as their target class, suffixed with 'Test' (eg. `Vigil.Patrons.PatronFactory`, `Vigil.Patrons.PatronFactoryTest`).

```csharp
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using Xunit;

namespace Vigil.MessageQueue.Commands
{
    public class CreatePatronCommandTest
    {
        public void CreatePatronCommand_Defaults_IsAnonymous()
        {
            CreatePatronCommand command = new CreatePatronCommand();

            Assert.False(command.IsAnonymous, "Default Value of CreatePatronCommand.IsAnonymous changed from 'false'.");
            Assert.Null(command.DisplayName);
            Assert.Null(command.PatronType);
        }

        [Fact]
        public void Validation_Requires_DisplayName_and_PatronType()
        {
            CreatePatronCommand command = new CreatePatronCommand();

            List<ValidationResult> validationResults = new List<ValidationResult>();
            Validator.TryValidateObject(command, new ValidationContext(command), validationResults, true);

            Assert.Contains(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(CreatePatronCommand.DisplayName)));
            Assert.Contains(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(CreatePatronCommand.PatronType)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(CreatePatronCommand.IsAnonymous)));
        }

        [Fact]
        public void Validation_On_DisplayName_Has_Maximum_String_Length()
        {
            CreatePatronCommand command = new CreatePatronCommand()
            {
                DisplayName = "This is a string with lots of letters appended.".PadRight(1000, 'A'),
                PatronType = "Invalid Type"
            };

            List<ValidationResult> validationResults = new List<ValidationResult>();
            Validator.TryValidateObject(command, new ValidationContext(command), validationResults, true);

            Assert.Contains(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(CreatePatronCommand.DisplayName)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(CreatePatronCommand.PatronType)));
            Assert.DoesNotContain(validationResults, vr => vr.MemberNames.Any(mn => mn == nameof(CreatePatronCommand.IsAnonymous)));
        }
    }
}
```

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
        [Fact]
        public void User_Can_Create_New_Patron()
        {
            var queue = new Mock<ICommandQueue>(MockBehavior.Strict);
            queue.Setup(q => q.QueueCommand(It.IsAny<ICommand>(), It.IsAny<IKeyIdentity>())).Verifiable();
            PatronFactory factory = new PatronFactory(queue.Object);

            FactoryResult result = factory.CreatePatron(new CreatePatronCommand()
            {
                DisplayName = "Test User",
                IsAnonymous = false,
                PatronType = "Test Account"
            });

            queue.VerifyAll();
            Assert.NotEqual(Guid.Empty, result.AffectedEntity.Id);
            Assert.Empty(result.ValidationResults);
        }

        [Fact]
        public void User_Cannot_Create_Patron_That_Fails_Validation()
        {
            var queue = new Mock<ICommandQueue>(MockBehavior.Strict);
            queue.Setup(q => q.QueueCommand(It.IsAny<ICommand>(), It.IsAny<IKeyIdentity>())).Verifiable();
            PatronFactory factory = new PatronFactory(queue.Object);

            FactoryResult result = factory.CreatePatron(new CreatePatronCommand());

            queue.Verify(q => q.QueueCommand(It.IsAny<ICommand>(), It.IsAny<IKeyIdentity>()), Times.Never);
            Assert.Null(result.AffectedEntity);
            Assert.NotEmpty(result.ValidationResults);
        }
    }
}
```