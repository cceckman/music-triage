# music-triage worklog

The first cut is Fine - it takes the intake directory and processes it. It
doesn't handle many of the corner cases, though, and reprocessing will take some
manual work.

Issues 1-7 have a bunch of features I'd like, but we probably need some
structural work before pushing any of them forward.

What are the steps in this flow?

1.  Scope: Discover the files to triage.

    This is the point at which we can re-run over the library, re-run over
    quarantine directory, etc.

2.  Plan: Determine what we _want_ to do; and why.

    This will wind up with some amount of serialization if we're trying to do
    duplicate detection. Maybe it's two-phase:

    - Generate target -> source mappings
    - resolve any duplicate mappings

    And hey - these work well concurrently; the "serial" step can be one
    goroutine, fed by many more goroutines (which are doing I/O).

3.  Execute: Create directories and move stuff around, if we're not in dry-run
    mode; or report the actions, if not. Clean up empty directories.

4.  Finalize: Produce report.

What data moves through this flow?

1.  Scope -> Plan: File paths.
2.  Plan:
    1.  Filepath -> (move action, reason)

        At this stage, even an error is an action (triage) and reason
        (encountered error).

        Move action is: (source, target, maybe-metadata)

        Maybe-metadata includes the metadata gleaned from from the source, if
        available - to be used in deduplicating actions

    2.  Set of (move action, reason) -> DAG of (action, reason), where action
        targets are unique

        This is the deduplication step, where we ensure that we aren't about to
        clobber ourselves.

        "Actions" here include "mkdir -p", "move", and "prune" (remove empty
        directories).

        "DAG" is a little strong / general - it's really a tree of `mkdir`
        inner nodes, with `mv` at the leaves, followed by `prune` ops with all
        other steps as prerequisites.

3.  Execute: DAG of (action, reason) -> (side-effects, errors, stats)

    Side effects are either dry-run side effects - "add a line to the report" -
    or actual side effects, "act on the filesystem".

    One way to accomplish this would be to report a shell script for the report,
    and execute it iff `!dry-run`. (Not optimally efficient, probably - Golang
    can do ops in parallel, in principle.)

    This is also where we count stats.

4.  Finalize: stats -> user output.

