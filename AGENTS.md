# Agent instructions

This is the public source of the `NuclearOption.Bindings` package, in the user's GitHub organization `scramd` (`scramd/nuclear-option-bindings`, transferred from the `clankagent` account on 2026-11-11; `clankagent` pushes with the organization role "All-repository maintain"). Work directly on `main` with reviewed commits; do not open pull requests for routine work.

- `src/NuclearBindings/` is the whole library. Its types stay `internal`: each mod compiles its own copy. Keep every file C# 8 compatible (netstandard2.1 default) with `#nullable enable` at the top.
- Nuclear Option types appear only in `InputContext.Blocked`, inside `#if !NUCLEAR_BINDINGS_GENERIC`. Keep it that way.
- Never commit game or Unity assemblies, decompiled code or anything from a game install. CompileCheck reads them from the developer's own install through MSBuild properties.
- Saved row text is a player-facing format. Do not rename `GamepadControl` values or change the row syntax without a conversion and a check.
- Before a commit, run `dotnet run --project tests/BindingChecks -c Release`. After a change to `BindingInput`, `BindingDrawer` or `InputBinding`, also run `tests/CompileCheck` in both modes.
- A release: raise `<Version>` in `NuclearOption.Bindings.csproj` and the README's PackageReference, add a CHANGELOG entry, tag `vX.Y.Z`, and attach the packed `.nupkg` to the GitHub release. Publishing the release runs `.github/workflows/publish.yml`, which pushes to nuget.org through Trusted Publishing (no API key; the policy names that file, and the repository variable `NUGET_USER` holds the nuget.org profile name). Publishing needs the user's approval.
- Logic and compile checks do not show that input reaches a mod in the game. Do not claim game behavior without a player's log or report.
