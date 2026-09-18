# Blazor TempData and Session Validation Sample

This is a small Blazor Web App that I created to validate the TempData and session parameter behavior described in [dotnet/aspnetcore#69139](https://github.com/dotnet/aspnetcore/issues/69139).

The application uses static server-side rendering (static SSR). Each navigation makes a new HTTP request, which makes it possible to observe when values are read, written back, retained, or removed.

## Prerequisites

- .NET 11 RC1 or a newer compatible .NET 11 SDK
- A web browser

The sample was tested with SDK `11.0.100-rc.1.26425.128`.

## Running the sample

Open a terminal in the project folder and run:

```powershell
dotnet run --launch-profile http
```

Then open:

```text
http://localhost:5274
```

Use the same browser tab while testing. Opening a new private window, clearing cookies, or restarting the application creates a different session.

The **Set fresh values** page generates a unique timestamp and short GUID. This seed is included in every value, so it is easy to confirm that a value shown on a later request belongs to the same test run.

## Build report

This repository was built and validated using the .NET 11 RC1 SDK. Captured build/run outputs from the validation machine are included below to reproduce the environment used during testing (run at 2026-09-17):

```text
.NET SDK:
 Version:           11.0.100-rc.1.26425.128
 Commit:            3551975be0

Host:
  Version:      11.0.0-rc.1.26425.128

--- dotnet restore ---
  Determining projects to restore...
  All projects are up-to-date for restore.

--- dotnet build ---
  BlazorStaticSSR -> D:\Validation\TempDataSessionValidation\BlazorStaticSSR\bin\Debug\net11.0\BlazorStaticSSR.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)
```

Include this output when submitting the validation evidence to make the SDK and build results explicit.



## Pages in the sample

| Page | Purpose |
|---|---|
| **Set fresh values** | Creates identifiable TempData and session values |
| **Display values** | Reads attributed values without modifying them |
| **Pass-through** | Makes a request without declaring or accessing the tested properties |
| **Alternate reader** | Reads the same keys through differently named properties |
| **Case-only reader** | Reads explicit keys using different casing |
| **Storage state** | Shows whether each key is present without consuming TempData |
| **Ordinary TempData read** | Compares an ordinary `ITempData.Get` read with an attributed read |
| **Duplicate TempData** | Deliberately renders two components using the same TempData key |
| **Duplicate session** | Deliberately renders two components using the same session key |
| **Distinct readers** | Shows the corrected version using separate A and B keys |
| **Assign null** | Assigns `null` to properties that currently contain values |

## Suggested validation flow

### 1. Explicit keys and property-name keys

1. Open **Set fresh values**.
2. Note the generated seed.
3. Compare the property and key columns.

`TempExplicit` and `SessionExplicit` use explicitly configured keys. `TempDefault` and `SessionDefault` don't specify a key, so their property names are used as the keys.

Open **Storage state** to confirm that `TempDefault` and `SessionDefault` are present.

### 2. Read the same key from another property

1. Open **Set fresh values** and note the seed.
2. Open **Alternate reader** without setting the values again.
3. Confirm that the differently named properties receive values containing the same seed.

This demonstrates that lookup is based on the configured key, not the reader's property name.

### 3. Read values on several requests

1. Open **Set fresh values**.
2. Select **Start request-count test**.
3. On **Display values**, select **Load display again** two or three times.
4. Confirm that all values still contain the original seed.

The display page leaves the bound properties unchanged. Their current values are written back at the end of each request, so the TempData values remain available instead of being consumed like an ordinary `ITempData.Get` read.

### 4. Pass through an unrelated page

1. Open **Set fresh values**.
2. Open **Pass-through**.
3. Select **Display values after pass-through**.
4. Confirm that all values still contain the original seed.

The pass-through page doesn't declare or access any of the tested properties.

### 5. Compare key casing

1. Open **Set fresh values**.
2. Open **Case-only reader**.
3. Compare the uppercase reader keys with the lowercase keys used to set the values.
4. Confirm that both values contain the original seed.

Both TempData and session keys matched case-insensitively in the tested build.

### 6. Duplicate key failures

Run the application in the Development environment so the full exception page is visible.

Open **Duplicate TempData**. The request intentionally fails with:

```text
InvalidOperationException: A callback is already registered for the TempData key 'validation-temp-duplicate'. Multiple components cannot use the same TempData key for multiple [SupplyParameterFromTempData] attributes.
```

Return to the home page and open **Duplicate session**. The request intentionally fails with:

```text
InvalidOperationException: A callback is already registered for the session key 'validation-session-duplicate'. Multiple components cannot use the same session key for multiple [SupplyParameterFromSession] attributes.
```

These failures are expected and don't stop the application.

### 7. Correct the duplicate keys

1. Open **Set fresh values** and note the seed.
2. Open **Distinct readers**.
3. Confirm that all four components render successfully:
   - `validation-temp-a` returns `TEMP-A-{seed}`.
   - `validation-temp-b` returns `TEMP-B-{seed}`.
   - `validation-session-a` returns `SESSION-A-{seed}`.
   - `validation-session-b` returns `SESSION-B-{seed}`.

The seed should be the same in all four values. The A and B prefixes should be different, proving that each component received its own value.

### 8. Assign null and reload

1. Open **Set fresh values**.
2. Optionally open **Storage state** and confirm that all four keys are present.
3. Open **Assign null**.
4. Select **Inspect key presence on next request**.
5. Open **Display values**.

Expected storage state after assigning `null`:

| Store | Expected key state | Displayed value |
|---|---|---|
| TempData | Key remains present | `<null>` |
| Session | Key is removed | `<null>` |

The next attributed reader displays `<null>` for both stores. The **Storage state** page is needed to distinguish a stored null TempData value from a removed session key.

### 9. Compare an ordinary TempData read

1. Open **Set fresh values**.
2. Open **Ordinary TempData read**.
3. Confirm that it displays the explicit TempData value.
4. Open **Storage state**.

The key read with `ITempData.Get` should be absent on the next request, while the unread TempData key should remain.

## Collecting evidence

For a clear recording:

1. Start by showing `dotnet --version` and the running application.
2. Record the seed every time **Set fresh values** is used.
3. Keep the browser URL visible.
4. Don't set fresh values between the steps of an individual scenario.
5. Capture the complete exception message for both duplicate-key pages.
6. Use **Storage state** when key presence matters.

Session values appear as Base64 text on **Storage state** because ASP.NET Core session stores serialized bytes. This is expected. Use **Display values** when you need to show the deserialized session strings.

## Results observed

On the tested .NET 11 RC1 build:

- Default keys used the property names.
- Differently named properties could read values through the same keys.
- Explicit-key lookup was case-insensitive for both attributes.
- Unchanged bound values survived several requests.
- The pass-through page didn't consume the values.
- Duplicate active keys threw `InvalidOperationException` and named the duplicated key.
- Components using distinct keys received independent values.
- Assigning `null` retained TempData keys with null values but removed session keys.
- An ordinary `ITempData.Get` read consumed the selected TempData key.

## Reference

- [Validation issue: dotnet/aspnetcore#69139](https://github.com/dotnet/aspnetcore/issues/69139)
- [ASP.NET Core Blazor state management](https://learn.microsoft.com/aspnet/core/blazor/state-management/server?view=aspnetcore-11.0#temporary-data-persistence)
- [Session and state management in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/app-state?view=aspnetcore-11.0)
