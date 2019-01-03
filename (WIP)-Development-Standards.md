# Development Standards
As a developer, use this as a checklist before submitting a PR. As a reviewer, use this as a reference when doing a review. 

Some of these standards are arbitrary. However, by having all developers comply with these arbitrary standards, we increase our overall code readability. Other standards are based on best practices and our own experience of what seems to work.

These only apply to C# code, but we encourage following the spirit of these rules for shaders.

## Hard Limits
These are hard rules that we will almost always make sure are followed in a PR.

1. All public classes and methods must be documented using /// comments. Parameters and return values must be documented and must actually be descriptive. Class documentation should describe the purpose of the class.
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
    1. Member variables must follow the convention "mVariableName"
    1. Method parameters must follow the convention "pParameterName"
    1. Static, non-constant members must follow the convention "sStaticName"
    1. Local variables must follow the convention "variableName"
    1. All constants or constant-like variables must follow the convention "CONSTANT_NAME"
1. Script folders should have no more than 10 scripts in a given folder. Reorganize, refactor, or add subfolders to meet this criteria.
1. Braces must always be used even when they are optional
1. Empty catch blocks are not allowed without a comment explaining why the catch is empty.
1. When accessing static members, you must use the class name and not an instance of the class - "ClassName.staticMethod" and not "someObject.staticMethod".

## Soft Limits
These guidelines are more up to judgement but are encouraged.

1. 

        