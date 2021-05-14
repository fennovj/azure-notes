# Data factory

Mainly some of the more complex side stuff, not how it works and what components there are etc.

## Azure Monitor

By default, Data Factory stores run-data for 45 days. You need Azure Monitor if you want to save it for longer. You can send it to: Storage Account, Event Hub, or Log Analytics.

Other than content of individual runs, the following metrics are tracked:

- ActivityRuns - The number of activities that were ran. Activities is stuff like webhook, copy data, stored proc, etc.
- PipelineRuns - Number of times a pipeline was ran (note multiple pipelines can run per trigger, etc)
- TriggerRuns - The amount of triggers (timer, manual, etc), so not a specific pipeline, just the amount of times a trigger was triggered
- SSIS stuff - used if you have custom ssis packages in ADF

To reiterate, a Pipeline consists of multiple activities. For example, 'Trigger Stored Procedure', 'Copy Data', 'For each loop' are all activities. A trigger is a run that responded to either a timer trigger or an event-based trigger.

## Integration Runtimes

- Azure
- Self-hosted
- SSIS

They kind of speak for themselves. For public-facing databases (with login/certificate obviously), you can just use Azure runtime. Self-hosted is for databases in private networks. There are also a few databases for which Azure IR does not have a driver (mainly SAP Hana) so you need a self-hosted IR.

The self-hosted IR is just a program you install on a server on the private network. It will make outbound connections to ADF and be managable from the ADF UI.

SSIS cannot do the same things like Copy Data tasks... It's really just for excecuting SSIS packages.

By default, the Azure IR *auto-resolves*, which means it picks a location by itself. It will try to select an IR that is close to the *sink* location. If the location is not detectable, it picks the ADF location.

If you have strict compliance requirements, you can create an Azure IR in a specific region, instead of using the default auto-resolving one. That will make sure it never accidentally runs in the wrong region.
