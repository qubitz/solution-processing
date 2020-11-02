# Solution Processing

## Overview

To put it briefly, *Solution Processing* is an event-based wrapper around
Unity`s sln/csproj post-processing calls.

Using more words, this library attempts to make persistent modifications of
.sln and .csproj files more accessible, readable, and maintainable by creating
a new API over the existing one. [Unity's current solution post-processing API]
is hidden, fragile, and unpolished.

Persistent modifications of .sln and .csproj files allow use of libraries
that require these modifications (i.e. Roslyn analyzers). Unity regenerates .sln and .csproj files
every time project dependencies change (which happens very often). Modifying
these files manually every time is exasperating, negating all the time saved by
using these libraries.

## How to use

To subscribe your custom delegates to any of the solution build events, inherit from
`SolutionProcessor` to get access to `ISolutionBuildEvents`. From there,
subscribe to any desired solution build events.

```c#
public class SolutionLogger : SolutionProcessor
{
    public override void RegisterSubscriptions(ISolutionBuildEvents events)
    {
        events.CSProjectGenerated += LogCompletedGeneration;
    }

    public void LogCompletedGeneration(IGeneratedFile file)
    {
        Console.WriteLine($"{file.File.Name} has been generated.")
    }
}
```

> Custom `SolutionProcessor`s must not add additional parameters to an
> overridden constructor. Doing otherwise will result in the instantiater
> being unable to construct the custom `SolutionProcessor` and an error will be
> thrown.

### Solution build events

* `CSProjectFilesPreGenerating`

  Occurs when the project files are about to be generated (.csproj and .sln
  files). It can occur even if the project files have already been generated.
  You can use this event's `IFileGenerationStatus` parameter to check and
  *control* the status.

* `SlnSolutionGenerated`

  Occurs when the .sln solution file has been generated. The generated contents
  can be modified by overwriting the string value or the file's contents via
  `FileInfo` within `IGeneratedFile`.

[Unity's current solution post-processing API]:https://github.com/Unity-Technologies/UnityCsReference/blob/61f92bd79ae862c4465d35270f9d1d57befd1761/Editor/Mono/AssetPostprocessor.cs#L70
