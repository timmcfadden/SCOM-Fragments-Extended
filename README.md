# SCOM Fragments Extended

Five additional SCOM Management Pack fragments for use with **VSAE** (Visual Studio Authoring Extensions) and **Silect MP Author**.

These are new fragments, not copies. They are written to drop straight into the workflow and conventions established by Kevin Holman's fragment library — same `##Token##` replacement scheme, same `%%` comment header that Silect MP Author reads, same ID and display-string patterns — so they can be used side by side with it.

## Credit

This work exists because of, and follows the conventions of, **[Kevin Holman](https://kevinholman.com)**.

- Original fragment library: **https://github.com/thekevinholman/FragmentLibrary**
- The authoring approach these fragments are built for: **[Authoring Management Packs – the fast and easy way using Visual Studio](https://kevinholman.com/2016/06/04/authoring-management-packs-the-fast-and-easy-way-using-visual-studio/)**

If you are new to fragment-based MP authoring, start with Kevin's library and that article. Everything about how these files are meant to be consumed is explained there.

Kevin's library is the upstream reference and is not redistributed here — this repository contains only the five fragments listed below. Note that the upstream repository does not carry an explicit license.

## The fragments

| Fragment | What it gives you |
|---|---|
| [`Combo.PowerShell.MultiInstance.Cookdown.Monitor.Performance.mpx`](Combo.PowerShell.MultiInstance.Cookdown.Monitor.Performance.mpx) | Class, scripted discovery, and **one shared PowerShell data source** feeding a threshold monitor and a performance collection rule for every discovered instance — a working cookdown example. |
| [`Monitor.TimedScript.PowerShell.REST.JSON.mpx`](Monitor.TimedScript.PowerShell.REST.JSON.mpx) | Two-state monitor that calls an HTTP(S) endpoint and validates a property inside the JSON response. |
| [`Combo.Class.Discovery.Certificate.Expiration.mpx`](Combo.Class.Discovery.Certificate.Expiration.mpx) | Certificate class, certificate-store discovery with subject / issuer / EKU criteria, and a three-state days-remaining monitor. |
| [`Monitor.EventLog.MissingEvent.TimeWindow.mpx`](Monitor.EventLog.MissingEvent.TimeWindow.mpx) | Detects the **absence** of an expected event within a rolling window, using the native SCOM missing-event monitor type. No scripting. |
| [`Monitor.TimedScript.PowerShell.ThreeState.ConsecSamples.mpx`](Monitor.TimedScript.PowerShell.ThreeState.ConsecSamples.mpx) | Reusable numeric PowerShell property-bag monitor with Healthy / Warning / Critical, supporting both "higher is worse" and "lower is worse". |

Every fragment carries its full documentation in its own header comment — replacement tokens, required references, defaults, override behaviour, threshold boundaries, and the design trade-offs. The tables below are a summary; the file is the authority.

### Replacement tokens

| Fragment | Tokens to replace |
|---|---|
| Cookdown | `##CompanyID##` `##AppName##` `##UniqueID##` `##BaseFolderPath##` `##IntervalSeconds##` `##ObjectName##` `##CounterName##` `##Threshold##` `##Samples##` |
| REST JSON | `##CompanyID##` `##AppName##` `##ClassID##` `##UniqueID##` `##URL##` `##ExpectedStatusCode##` `##JsonPropertyPath##` `##ExpectedValue##` `##IntervalSeconds##` `##RequestTimeoutSeconds##` `##ConsecutiveFailures##` |
| Certificate | `##CompanyID##` `##AppName##` `##UniqueID##` `##StoreLocation##` `##StoreName##` `##SubjectMatch##` `##IssuerMatch##` `##EnhancedKeyUsageMatch##` `##IntervalSeconds##` `##WarningDaysRemaining##` `##CriticalDaysRemaining##` |
| Missing event | `##CompanyID##` `##AppName##` `##ClassID##` `##LogName##` `##EventID##` `##EventSource##` `##IntervalSeconds##` |
| Three-state | `##CompanyID##` `##AppName##` `##ClassID##` `##UniqueID##` `##IntervalSeconds##` `##Direction##` `##WarningThreshold##` `##CriticalThreshold##` `##NumSamples##` |

### Required references

| Fragment | Aliases |
|---|---|
| Cookdown | `System` `SC` `Windows` `Health` `Perf` `MSDL` |
| REST JSON | `System` `Windows` `Health` |
| Certificate | `System` `Windows` `Health` |
| Missing event | `Windows` `Health` |
| Three-state | `System` `Windows` `Health` |

## Using them

1. Copy the `.mpx` file into your VSAE management pack project (or import it as a fragment in Silect MP Author).
2. Replace every `##Token##` listed in the file's header. `##CompanyID##`, `##AppName##` and `##UniqueID##` must contain no spaces or special characters.
3. Add the references the header lists, with the aliases exactly as named — the fragments reference them by alias (`Windows!`, `Health!`, and so on).
4. Build.

Each fragment's header explains which parameters are overrideable and what overriding them actually does.

## Notes worth reading before you deploy

- **Cookdown is per agent.** The shared data source in the cookdown fragment runs once per interval *per health service*. It never shares a script execution between different agents. The fragment header spells out exactly which overrides preserve sharing and which silently split it.
- **The missing-event monitor is a rolling fixed-length window**, not a calendar deadline. It cannot express "the job must have finished by 06:00", and the header says so rather than implying otherwise.
- **Consecutive samples are not hysteresis.** The three-state monitor delays a state change; it does not implement separate recovery thresholds. The header documents what you would have to add to get real hysteresis.
- **Collection failures are never reported as healthy.** The certificate discovery refuses to submit an empty snapshot when it cannot read the store, because an empty snapshot deletes live objects. The scripted monitors report a failure state with a distinguishable reason rather than defaulting to Success.
- **The REST monitor keeps TLS certificate validation enabled**, never embeds credentials, and keeps the response body out of alerts and events. It targets Windows PowerShell 5.1 as hosted by the SCOM agent — it does not assume PowerShell 7.

## Validation status

Completed:

- All five parse as XML and **verify with the SCOM SDK** against the sealed OM2019 reference management packs, both individually and merged together into a single management pack.
- Replacement-token lists, reference aliases, internal IDs, display strings, alert-parameter mappings, override selectors, and configuration propagation were checked programmatically against actual usage.
- External module and monitor-type IDs and their configuration schemas were confirmed against the sealed reference management packs, not from memory.
- The PowerShell in each fragment was extracted from the `.mpx` and executed in Windows PowerShell 5.1 against representative healthy, warning, critical and failure inputs — including threshold boundary values, non-numeric and missing data, unreadable certificate stores, a genuinely expired certificate, and a local HTTP server returning each distinct failure mode.

Not yet verified, because it requires a live management group:

- Cookdown execution counts on a real agent.
- Missing-event detection timing, startup grace, and behaviour across an agent restart.
- The consolidator alert-context XPaths at runtime.
- Performance data landing in both the operational database and the data warehouse.
- Run As profile behaviour for the REST monitor.
- Discovery churn as a certificate is renewed and the old one removed.

Treat these as working fragments that have been statically verified and script-tested, and test them in a lab before production — the same advice that applies to any fragment.
