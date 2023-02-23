---
title: A Word On Factory Results and Contracts
category: Vigil Journey
tags:
- vigil
- factory
- codecontracts
date: 2016-09-17
---

For simplicity, the `PatronFactory`'s `CreatePatron` method just returned an `IKeyIdentity`. This was a simple interface that requires an `Id` of type `Guid`. When the creation method was always assumed to work correctly, this was fine - I don't have to worry about validation checking, data errors, or anything that might cause a failure. However, in the real world, I need to be aware of these things. As such, the first refactor that I'm going to perform is to change the return type to a multi-purpose class.


### FactoryResult

```csharp
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace Vigil.Domain
{
    public class FactoryResult
    {
        public ICollection<ValidationResult> ValidationResults { get; protected set; }
        public IKeyIdentity AffectedEntity { get; protected set; }

        public FactoryResult(ICollection<ValidationResult> validationResults)
        {
            ValidationResults = validationResults;
        }
        public FactoryResult(IKeyIdentity affectedEntity)
        {
            AffectedEntity = affectedEntity;
        }
    }
}
```

The `PatronFactory` will now return this class. If there were problems, then the `ValidationResults` collection will contain elements; if all went well, then `IKeyIdentity` should not be null. Tests were updated to reflect this, and now I am ready to add validation ([next post](/posts/2016/user-cannot-create-patron-that-fails-validation/)).

### A Quick Note About Contracts

Currently, Code Contracts is not supported for .NetCore and is not coming to C# 7.0, which makes me very sad - it is also the number one requested feature for [C# on the Visual Studio uservoice](https://visualstudio.uservoice.com/forums/121579-visual-studio-2015/suggestions/2320188-add-non-nullable-reference-types-in-c). While the primary use of Design by Contract was for eliminating null references, there were still plenty of other places it was extremely useful. Especially with `Guid`s becoming the primary source of identifiers, it was very helpful to be able to Require that a Guid could not be Empty, or that a value was in a given range.

So, it is with a sad heart that I have removed Contracts from this project entirely. I will start putting in null reference checks where appropriate, but for the most part, I will do what I can to find an alternative solution.