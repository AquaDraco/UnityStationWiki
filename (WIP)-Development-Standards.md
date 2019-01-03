As a developer, use this as a checklist before submitting a PR. As a reviewer, use this as a reference when doing a review. 

Some of these standards are arbitrary. However, by having all developers comply with these arbitrary standards, we increase our overall code readability. Other standards are based on best practices and our own experience of what seems to work.

These only apply to C# code, but we encourage following the spirit of these rules for shaders.

## Hard Limits
These are hard rules that we will almost always make sure are followed in a PR.

1. All public classes, structs, properties, and methods must be documented using /// comments. Parameters and return values must be documented and must actually be descriptive. Class documentation should describe the purpose of the class.
1. Each script file must have at most one publicly-exported class or struct. Do not define multiple public classes / structs in one script file.
1. All code must be formatted according to the style in the top-level .editorconfig. For reference (or in case .editorconfig is not yet updated with these), here's the style conventions:
    1. Indent using tabs.
    1. Line endings must always be checked into git as Unix-style (LF). If you are on Windows, ensure that you have configured git to check out windows style but commit unix-style line endings (git config --global core.autocrlf true)
    1. Lines of code must not exceed 120 characters
    1. Curly braces must always be on a line by themselves:
    ````
    if (condition)
    {
        ....
    }
    else
    {
        ....
    }
    ````
1. Variable naming conventions. These help to identify the scope of a variable without needing to look up its definition (i.e. to be able to tell if it's a class member or a parameter).
    1. Fields must use camel case starting with an m - "mFieldName"
    1. Method parameters must use camel case starting with a p - "pParameterName"
    1. Static, non-constant members must use camel case starting with an s - "sStaticName"
    1. Local variables must use camel case - "variableName"
    1. All constants or constant-like variables must use uppercase snake case - "CONSTANT_NAME"
1. Properties, classes, structs, and methods should use camel case with the first letter capitalized - "SomeName".
1. Script folders should have no more than 10 scripts in a given folder. Reorganize, refactor, or add subfolders to meet this criteria.
1. Braces must always be used even when they are optional
1. Empty catch blocks are not allowed without a comment explaining why the catch is empty.
1. When accessing static members, you must use the class name and not an instance of the class - "ClassName.staticMethod" and not "someObject.staticMethod".

## Soft Limits
These guidelines are more up to the reviewer's judgement but are encouraged.

1. Any TODO comments in the code under review must be accompanied by a corresponding issue in the issue tracker. TODO comments represent tech debt and should be taken seriously.
1. Fields should not be public. Use properties (or automatic properties) instead or add logic inside the class rather than exposing its state.
1. Fields should not be protected unless they are needed by the child class.
1. Avoid many levels of nested indentation, almost certainly no more than 7, preferable no more than 3. You can very easily solve this by encapsulating logic inside a block by putting it into its own method.
1. Try to keep individual class files small. This decreases the amount of context necessary to understand a portion of code. Shoot for less than 500 actual lines of code (ignoring comments / blank lines). Use refactoring, design patterns, and other techniques to try to keep them small.
1. When deciding what type to use, strings should be used only as a last resort. Prefer other types, such as enums, numeric types, custom classes, etc...if they are more appropriate.
1. Avoid using getters and setters. Follow the philosophy "tell, don't ask". Instead of getting and setting values within an object, tell the object to do something with its own data. Treat a class as a folder for organizing data and the logic that operates on that data.
1. Most string or numeric literals should be constants.
1. Don't define constants that have the same value in multiple places. There should only ever be one place you need to change if you ever need to change a constant's value.
1. Local variables should be declared near where they are used, not at the beginning of their containing block.

        