# TempData and session parameter validation

Static SSR sample for [dotnet/aspnetcore#69139](https://github.com/dotnet/aspnetcore/issues/69139), targeting .NET 11 RC1.

## Run

```powershell
dotnet run --launch-profile http
```

Open `http://localhost:5274`. Use the home page's ordered recording sequence. Always start a comparison with **Set fresh values**, which creates an identifiable timestamp/GUID seed.

## Scenario coverage

| Requirement | Route |
|---|---|
| Explicit and property-name default keys | `/set-values`, `/display-values` |
| Next three requests and untouched writeback | `/display-values` |
| Request declaring neither attribute | `/passthrough` |
| Different property name reading the same key | `/alternate-reader` |
| Case-only key comparison | `/case-reader` |
| Duplicate active TempData key | `/duplicate-tempdata` |
| Duplicate active session key | `/duplicate-session` |
| Corrected distinct component keys | `/distinct-readers` |
| Assign null and inspect key presence | `/clear-values`, `/storage-state` |
| Ordinary TempData consumption comparison | `/ordinary-tempdata-read` |

The duplicate routes intentionally fail with `InvalidOperationException`. Run with the Development environment to record the verbatim developer exception page.

## Validated observations

Validated on .NET SDK `11.0.100-rc.1.26425.128`:

| Check | Observed result |
|---|---|
| Three consecutive attributed reads | Both TempData and session values remained visible because each unchanged property was written back |
| Pass-through request | Neither store was consumed |
| Different property names with explicit original keys | Both stores returned the original values |
| Explicit keys differing only by case | Both stores matched case-insensitively |
| Duplicate TempData key | HTTP 500 with `InvalidOperationException` naming `validation-temp-duplicate` |
| Duplicate session key | HTTP 500 with `InvalidOperationException` naming `validation-session-duplicate` |
| Corrected distinct keys | All four components independently received their seeded value |
| Assigning null | TempData retained present keys with null values; session removed the keys |
| Ordinary `ITempData.Get` | The read TempData key was absent on the next request while the unread key remained |

## Evidence checklist

Record one continuous browser video showing:

1. The SDK version from `dotnet --version` and the running app.
2. A fresh seed and the next three `/display-values` requests.
3. A fresh seed, `/passthrough`, then `/display-values`.
4. Fresh seeds before `/alternate-reader` and `/case-reader`.
5. Both duplicate-key exception pages, including the complete exception message.
6. A fresh seed followed by `/distinct-readers`.
7. A fresh seed, `/clear-values`, and `/storage-state`.
8. A fresh seed, `/ordinary-tempdata-read`, then `/storage-state`.

The [Blazor state-management documentation](https://learn.microsoft.com/aspnet/core/blazor/state-management/server?view=aspnetcore-11.0#temporary-data-persistence) documents TempData as data that is discarded after it is read and session as data retained while the session is maintained. It describes both attributes and key naming, but doesn't provide a direct selection guide or explicitly explain request-end attributed-property writeback lifetime. Capture observed behavior rather than assuming a fixed request count.