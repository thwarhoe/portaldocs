{"gitdown": "contents"}

# Performance Overview

Portal performance from a customers perspective is seen as all experiences throughout the product. 
As an extension author you have a duty to uphold your experience to the performance bar at a minimum.

| Area      | 95th Percentile Bar | Telemetry Action         | How is it measured? |
| --------- | ------------------- | ------------------------ | ------------------- |
| Extension | See Power BI        | ExtensionLoad            | The time it takes for your extension's home page to be loaded and initial scripts, the `initialize` call to complete within your Extension defintion file  |
| Blade     | See Power BI        | BladeFullReady           | The time it takes for the blade's `onInitialize` or `onInputsSet` to resolve and all the parts on the blade to become ready |
| Part      | See Power BI        | PartReady                | Time it takes for the part to be rendered and then the part's OnInputsSet to resolve |
| WxP       | See Power BI        | N/A                      | An overall experience score, calculated by weighting blade usage and the blade full ready time |

## Extension performance

Extension performance effects both Blade and Part performance, as your extension is loaded and unloaded as and when it is required.
In the case a user is visiting your resource blade for the first time, the Fx will load up your extension and then request the view model, consequently your Blade/Part
performance is effected.
If the user were to browse away from your experience and browse back before your extension is unloaded obviously second visit will be faster, as they don't pay the cost
of loading the extension.

## Blade performance

Blade performance is spread across a couple of main areas:

1. Blade's constructor
1. Blade's onInitialize or onInputsSet
1. Any Parts within the Blade become ready

All of which are encapsulated under the one BladeFullReady action.

### BladePerformanceIncludingNetwork()

In the case of the `BladePerformanceIncludingNetwork` function, we sample 1% of traffic to measure the number of network requests that are made throughout their session. Within the fucntion we will correlate the count of any network requests made when the user is loading a given blade. It does not impact the markers that measure performance. That said a larger number of network requests will generally result in slower performance.
The subtle difference with the standard 'BladeFullReady' marker is that if the blade is opened up within a resource menu blade we will attribute the time it takes to resolve the `getMenuConfig` promise as the resource menu blade is loaded to the 95th percentile of the 'BladeFullReady' duration. There is a known calculation issue with the current approach as the 95th percentile is reported as the summation of both the 95th percentiles for 'BladeFullReady' and the `getMenuConfig` call. In most cases this is insignificant as the `getMenuConfig` 95th percentile is < 10ms due it being static, if you are drastically affected by this then please make your menu statically defined.

## Part performance

Similar to Blade performance, Part performance is spread across a couple of areas:

1. Part's constructor
1. Part's onInitialize or onInputsSet

All of which are encapsulated under the one PartReady action.

## WxP score

The WxP score is a per extension Weight eXPerience score (WxP). It is calculated by the as follows:

```txt

WxP = (BladeViewsMeetingTheBar * 95thPercentileBar) / ((BladeViewsMeetingTheBar * 95thPercentileBar) + ∑(BladeViewsNotMeetingTheBar * ActualLoadTimePerBlade))

```

| Blade   | 95th Percentile Times | Usage Count | Meets 95th Percentile Bar? |
| ------- | --------------------- | ----------- | -------------------------- |
| Blade A | 1.2                   | 1000        | Yes                        |
| Blade B | 5                     | 500         | No                         |
| Blade C | 6                     | 400         | No                         |

```txt

WxP = (BladeViewsMeetingTheBar * 95thPercentileBar) / ((BladeViewsMeetingTheBar * 95thPercentileBar) + ∑(BladeViewsNotMeetingTheBar * ActualLoadTimePerBlade)) %
    = (4 * 1000) / ((4 * 1000) + ((5 * 500) + (6 * 400))) %
    = 44.94%

```

As you can see the model penalizes the number of views which don’t meet the bar and also the count of those.

## How to assess your performance

There is two methods to assess your performance:

1. Visit the IbizaFx provided PowerBi report*
1. Run Kusto queries locally to determine your numbers

    (*) To get access to the PowerBi dashboard reference the [Telemetry onboarding guide][TelemetryOnboarding], then access the following [Extension performance/reliability report][Ext-Perf/Rel-Report]

The first method is definitely the easiest way to determine your current assessment as this is maintained on a regular basis by the Fx team.
You can, if preferred, run queries locally but ensure you are using the Fx provided Kusto functions to calculate your assessment.

# Performance Frequently Asked Questions (FAQ)

## My Extension 'load' is above the bar, what should I do

1. Profile what is happening in your extension load
1. Are you using obsolete bundles? If yes, remove them.
1. Are you using the Portal's ARM token? If no, verify if you can use the Portal's ARM token and if yes, follow: [Using the Portal's ARM token](http://NEED_LINK.com)
1. Are you on the hosting service? If no, migrate to the hosting service: [Hosting service documentation](http://NEED_LINK.com)
1. Do you see any waterfalling or serialized bundle requests? If yes, ensure you have the proper bundling hinting in place. [Optimize bundling](http://NEED_LINK.com) 

## My Blade 'FullReady' is above the bar, what should I do

1. Assess what is happening in your Blades's `onInitialize` or constructor and `onInputsSet`.
1. Can that be optimized?
1. If there are any AJAX calls;
    1. Ensure they called using the batch api
    1. Wrap them with custom telemetry and ensure they you aren't spending a large amount of time waiting on the result.
1. How many parts are on the blade?
    - If there's only a single part, if you're not using a no-pdl blade or a `<TemplateBlade>` migrate your current blade over.
    
## My Part 'Ready' is above the bar, what should I do

1. Assess what is happening in your Part's `onInitialize` or constructor and `onInputsSet`.
1. Can that be optimized?
1. If there are any AJAX calls;
    1. Ensure they called using the batch api
    1. Wrap them with custom telemetry and ensure they you aren't spending a large amount of time waiting on the result.

## My WxP score is below the bar, what should I do

Using the [Extension performance/reliability report][Ext-Perf/Rel-Report] you can see the WxP impact for each individual blade. Although given the Wxp calculation,
if you are drastically under the bar its likely a high usage blade is not meeting the performance bar, if you are just under the bar then it's likely it's a low usage
blade which is not meeting the bar.

## Is there any way I can get further help

Sure! Book in some time in the Azure performance office hours.

- __When?__  Wednesdays from 1:00 to 3:30
- __Where?__ B42 (Conf Room 42/46)
- __Contacts:__ Sean Watson (sewatson)
- __Goals__
    - Help extensions to meet the performance bar
    - Help extensions to measure performance 
    - Help extensions to understand their current performance status
- __How to book time__: Send a meeting request with the following
    - TO: sewatson;
    - Subject: YOUR_EXTENSION_NAME: Azure performance office hours
    - Location: Conf Room 42/46 (It is already reserved)


# Performance best practices

## Operational best practices

- Enable performance alerts
    - To ensure your experience never regresses unintentionally, you can opt into configurable alerting on your extension, blade and part load times. See [performance alerting](index-portalfx-extension-monitor.md#performance) 
- Move to [hosting service](portalfx-extension-hosting-service.md#extension-hosting-service)
    - If you are not on the hosting service ensure;
        1. Homepage caching is enabled
        1. Persistent content caching is enabled
        1. Compression is enabled
        1. Your service is efficiently geo-distributed (Note we have seen better performance from having an actual presence in a region vs a CDN) 
- Compression (Brotli)
    - Move to V2 targets to get this by default
- Remove controllers 
    - Don't proxy ARM through your controllers

## Coding best practices

- Reduce network calls
    - Ideally 1 network call per blade
    - Utilise `batch` to make network requests, see below for an example of how to use batching

    ```ts
    import { batch } from "Fx/Ajax";

    return batch<WatchResource>({
        headers: { accept: applicationJson },
        isBackgroundTask: false, // Use true for polling operations​
        setAuthorizationHeader: true,
        setTelemetryHeader: "Get" + entityType,
        type: "GET",
        uri: uri,
    }, WatchData._apiRoot).then((response) => {
        const content = response && response.content;
        if (content) {
            return convertFromResource(content);
        }
    });
    ```

- Remove automatic polling
    - If you need to poll, only poll on the second request and ensure `isBackgroundTask: true` in the batch call
- Optimize bundling (Avoiding the waterfall)

    ```typescript
    /// <amd-bunding root="true" priority="0" />

    import ClientResources = require("ClientResources");
    ```

- Remove all dependencies on obsoleted code
    - Loading any required obsoleted bundles is a blocking request during your extension load times
    - See https://aka.ms/portalfx/obsoletebundles for further details
- Use the Portal's ARM token
- Don't use old PDL blades composed of parts: [hello world template blade](portalfx-no-pdl.md#building-a-hello-world-template-blade-using-decorators)
    - Each part on the blade has it's own viewmodel and template overhead, switching to a no-pdl template blade will mitigate that overhead
- Use the latest controls available: see https://aka.ms/portalfx/playground
    - This will minimise your observable usage
    - The newer controls are AMD'd reducing what is required to load your blade
- Remove Bad CSS selectors
    - Build with warnings as errors and fix them
    - Bad CSS selectors are defined as selectors which end in HTML elements for example `.class1 .class2 .class3 div { background: red; }`
        - Since CSS is evaluated from right-to-left the browser will find all `div` elements first, this is obviously expensive
- Fix your telemetry
    - Ensure you are returning the relevant blocking promises as part of your initialization path (`onInitialize` or `onInputsSet`)
    - Ensure your telemetry is capturing the correct timings

## General best practices

- Test your scenarios at scale
    - How does your scenario deal with 100s of subscriptions or 1000s of resources?
    - Are you fanning out to gather all subscriptions, if so do not do that.
        - Limit the default experience to first N subscriptions and have the user determine their own scope.
- Develop in diagnostics mode
    - Use https://portal.azure.com?trace=diagnostics to detect console performance warnings
        - Using too many defers
        - Using too many ko.computed dependencies
- Be wary of observable usage
    - Try not to use them unless necessary
    - Don't aggreesively update UI-bound observables
        - Accumulate the changes and then update the observable
        - Manually throttle or use `.extend({ rateLimit: 250 });` when initializing the observable
- Run portalcop to identify and resolve common performance issues


# Performance profiling 

## How to profile your scenario

1. Open a browser and load portal using https://portal.azure.com/?clientoptimizations=bundle&feature.nativeperf=true​
    - `clientOptimizations=bundle` will allow you to assess which bundles are being downloaded in a user friendly manner
    - `feature.nativeperf=true` will expose native performance markers within your profile traces, allowing you to accurately match portal telemetry markers to the profile
1. Go to a blank dashboard​
1. Clear cache (hard reset) and reload the portal​
1. Use the browsers profiling timeline to throttle both network and CPU, this best reflects the 95th percentile scenario, then start the profiler
1. Walk through your desired scenario
    - Switch to the desired dashboard
    - Deep link to your blade, make sure to keep the feature flags in the deep link. Deeplinking will isolate as much noise as possible from your profile
1. Stop the profiler

## Idenitifying common slowdowns

1. Blocking network calls
1. Waterfalling bundling
1. Heavy rendering and CPU from overuse of UI-bound observables

## Verifying a change

To correctly verify a change you will need to ensure the before and after are instrumented correctly with telemetry. Without that you cannot truly verify the change was helpful.
We have often seen what seems like a huge win locally transition into a smaller win once it's in production, we've also seen the opposite occur too.
The main take away is to trust your telemetry and not profiling, production data is the truth. 


[TelemetryOnboarding]: <portalfx-telemetry-getting-started.md>
[Ext-Perf/Rel-Report]: <http://aka.ms/portalfx/dashboard/extensionperf>