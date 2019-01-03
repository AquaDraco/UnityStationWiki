As we add more features and grow our codebase, we all want to ensure that we can still understand the code. If we let our codebase get messy, it will take longer to add new features and fix bugs, and it'll get even more difficult as the codebase grows. It would be awesome to build a game as complex as SS13 that is also easy to keep improving upon. Remember that each line of code you add now will be read by many other fellow developers in the future (including you!), so writing code that's easy to understand has a very positive impact on the project. 

This page provides some suggestions you can use to help us achieve that - to keep our code "clean". 

These are NOT rules, and reviewers don't expect developers to be able to follow all of these suggestions. When a reviewer looks at your PR, part of their review will involve thinking about code cleanliness. They will work with you to figure out if / what changes can be made to the PR to help with code cleanliness. They might suggest some items from this list or something from their own experience. As you submit more and more PRs, the reviewers you collaborate with should help you develop a better sense of how to write clean code until it becomes second nature.

TODO: Code examples, lots of them.
TODO: Split into subsections so it's not one huge list

# Code Style
1. Document public stuff using /// comments. This includes public classes, structs, properties, fields, and methods. Parameters and return values should be descriptive. Class documentation should describe the purpose of the class.
Here's good examples of how to document each of these: TODO 
1. Use a separate .cs file for each public class or struct. Avoid defining multiple public classes / structs in one script file.
1. Format code according to our standard style. Your IDE will be able to apply some of these automatically by reading this from our .editorconfig file. For reference (or in case .editorconfig can't support these), here's the style conventions:
    1. Indent using tabs rather than spaces.
    1. When checking in code, try to ensure it uses Unix-style (LF) line endings. If you're on OSX or Linux, you don't need to do anything. If you are on Windows, ensure that you have configured git to check out Windows style but commit Unix-style line endings using the command `git config --global core.autocrlf true` or configuring this using your git GUI of choice.
    1. Avoid long lines. Break up lines of code that are longer than 120 characters. Alternatively, try to refactor the statement so it doesn't need to be so long!
TODO: give two Code examples of breaking up and refactoring

    1. Curly braces should always be on a line by themselves:
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
1. All variables except for constants / constant-like ones should use camelCase (with no prefix letter).
TODO: Code example
1. All constants or constant-like variables should use uppercase snake case - "CONSTANT_NAME"
TODO: Code example
1. Properties, classes, structs, and methods should use camel case with the first letter capitalized - "SomeName".
TODO: Code example
1. Any TODO comments added in a PR should be accompanied by a corresponding issue in the issue tracker. A TODO comment means that there's something we need to remember to do, so we need to put it in the issue tracker to make sure we remember.
1. Script folders should have no more than 10 scripts in a given folder. Reorganize, refactor, or add subfolders as needed to avoid folders getting too large.
1. Braces should always be used even when they are optional
TODO: Code example
1. Local variables should be declared near where they are used, not at the beginning of their containing block.
TODO: Code example of the correct way with the bad way commented out

# Code Design
1. Follow the philosophy "tell, don't ask". TELL a class to do something rather than ASKING it for some data and operating on that data. Treat a class as a folder for grouping together data and the logic that operates on that data.
TODO: Code example of tell, don't ask.
1. Empty catch blocks should be rare, because it means something exceptional happened and we aren't doing anything about it. At the very least, a catch block should probably log an error or warning. If it REALLY needs to be empty should have a comment explaining why the catch is empty.
TODO: Code example of catch block logging an error message and catch block with a comment
1. Avoid many levels of nested indentation, almost certainly no more than 7, preferably no more than 3. You can solve this by taking deeply-nested logic and putting it into a descriptively-named private method.
1. Try to keep individual .cs files small. Shoot for less than 500 actual lines of code (ignoring comments / blank lines). You can use refactoring, design patterns, and other techniques to try to make them small by splitting logic up into other .cs files / classes.
1. When deciding what type to use, strings should be used only as a last resort. Prefer other types, such as enums, numeric types, custom classes, etc...if they are more appropriate.
1. Most string or numeric literals should be constants.
1. Don't define constants that have the same value in multiple places. There should only ever be one place you need to change if you ever need to change a constant's value.
        