# TempData and Session Validation

This repository contains the sample application and evidence I used to validate the TempData and session parameter scenarios from [dotnet/aspnetcore#69139](https://github.com/dotnet/aspnetcore/issues/69139).

The validation focuses on key selection, duplicate-key handling, value lifetime, null assignment, and request-end writeback for:

- `[SupplyParameterFromTempData]`
- `[SupplyParameterFromSession]`

## Repository contents

```text
TempDataSessionValidation/
|-- BlazorStaticSSR/   Blazor Web App used for the validation
|-- Evidence/          Recorded validation evidence and report
`-- README.md          Repository overview
```

The application uses Blazor static server-side rendering. Every navigation makes a new HTTP request, allowing the stored values to be followed across requests.

For detailed test instructions, expected results, and an explanation of every page, see the [sample README](BlazorStaticSSR/README.md).

## Requirements

- .NET 11 RC1 or a newer compatible .NET 11 SDK
- A web browser

The sample was validated with .NET SDK `11.0.100-rc.1.26425.128`.

## Run the sample

Open:

```powershell
cd BlazorStaticSSR
dotnet run --launch-profile http
```

Open:

```text
http://localhost:5274
```

Use the same browser tab during a scenario so that the session cookie is retained. Start each independent scenario from **Set fresh values** and note the generated seed.

## Build report

The sample was built and tested on the validation machine using .NET SDK 11 RC1. Captured output from the validation run is shown below (run at 2026-09-17):

```text
.NET SDK:
 Version:           11.0.100-rc.1.26425.128
 Commit:            3551975be0

Host:
  Version:      11.0.0-rc.1.26425.128

global.json file:
  D:\Validation\TempDataSessionValidation\BlazorStaticSSR\global.json

--- dotnet restore ---
  Determining projects to restore...
  All projects are up-to-date for restore.

--- dotnet build ---
  BlazorStaticSSR -> D:\Validation\TempDataSessionValidation\BlazorStaticSSR\bin\Debug\net11.0\BlazorStaticSSR.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)
```

Include these outputs in your submission to the review to prove the exact SDK and a successful build with the tested RC1 SDK.

## Scenarios covered

From the repository root:

```powershell
cd BlazorStaticSSR
dotnet run --launch-profile http
```

Open:

```text
http://localhost:5274
```

Use the same browser tab during a scenario so that the session cookie is retained. Start each independent scenario from **Set fresh values** and note the generated seed.

## Scenarios covered

- Explicit keys and keys defaulted from property names
- Reading the same key through a differently named property
- Case-insensitive explicit-key matching
- Repeated attributed reads and request-end writeback
- A pass-through request that doesn't access the stored properties
- Duplicate TempData and session key exceptions
- Corrected components using distinct keys
- Assigning null and checking key presence separately
- Ordinary `ITempData.Get` consumption

## Evidence

The [Evidence](Evidence/) folder contains the validation report and recordings.

### Report

- [TempData and Session Key Lifetime Validation Report](Evidence/TempDataAndSession_KeyLifetimeValidationReport.docx)

### Static SSR recordings

- [Default key sharing](Evidence/Static%20SSR/DefaultKeySharing.mp4)
- [Distinct key isolation](Evidence/Static%20SSR/DistinctKeyIsolation.mp4)
- [Duplicate key conflict](Evidence/Static%20SSR/DuplicateKeyConflict.mp4)
- [Explicit and default key comparison](Evidence/Static%20SSR/ExplicitDefaultKeyComparison.mp4)
- [Idle request persistence](Evidence/Static%20SSR/IdleRequestPersistence.mp4)
- [Navigation round-trip persistence](Evidence/Static%20SSR/NavigationRoundTripPersistence.mp4)
- [Null assignment retention](Evidence/Static%20SSR/NullAssignmentRetention.mp4)
- [Null value persistence](Evidence/Static%20SSR/NullValuePersistence.mp4)
- [Sequential reload retention](Evidence/Static%20SSR/SequentialReloadRetention.mp4)
- [Shared key property mapping](Evidence/Static%20SSR/SharedKeyPropertyMapping.mp4)
- [Unique key value isolation](Evidence/Static%20SSR/UniqueKeyValueIsolation.mp4)
- [Unused page value retention](Evidence/Static%20SSR/UnusedPageValueRetention.mp4)

## Results

The tested .NET 11 RC1 build behaved as follows:

- A missing explicit key defaulted to the property name.
- A differently named property could read the same stored key.
- TempData and session explicit keys matched case-insensitively.
- Unchanged bound values survived repeated requests because they were written back.
- An unrelated page didn't consume the values.
- Duplicate active keys threw `InvalidOperationException` and named the conflicting key.
- Components with distinct keys received independent values.
- Assigning null retained a TempData key with a null value but removed the session key.
- An ordinary `ITempData.Get` read consumed the selected TempData key.

## References

- [Validation issue: dotnet/aspnetcore#69139](https://github.com/dotnet/aspnetcore/issues/69139)
