---
layout: page_v2
title: Prebid 1.0 Publisher API Changes
description: Description of the changes to the publisher facing API for Prebid 1.0
top_nav_section: dev_docs
nav_section: reference

sidebarType: 1
---



# Prebid 1.0 Publisher API Changes
{:.no_toc}

This document describes the changes to the Publisher API for Prebid.js version 1.0.

* TOC
{:toc}

## Removed Functions and Variables

The following functions and variables were removed as of 1.0:

- All `pbjs._*` variables, including:
  - `pbjs._winningBids`
  - `pbjs._bidsReceived`
  - `pbjs._bidsRequested`
  - `pbjs._adUnitCodes`
  - `pbjs._adsReceived`
  - `pbjs.cbTimeout`
- `pbjs.addCallback` and `pbjs.removeCallback` in favor of the [onEvent API](/dev-docs/publisher-api-reference/onEvent.html)
- `pbjs.allBidsAvailable`
- `pbjs.buildMasterVideoTagFromAdserverTag` in favor of [`pbjs.adServers.dfp.buildVideoUrl`](/dev-docs/publisher-api-reference/adServers.dfp.buildVideoUrl.html)
- `adUnit.sizeMapping` in favor of [`pbjs.setConfig({sizeConfig:[ ... ]})`]({{site.baseurl}}/dev-docs/prebid-1.0-API.html#size-mapping-changes)

Other methods were removed as part of the [new `setConfig` API](/dev-docs/publisher-api-reference/setConfig.html) - for details, see [the section below describing the new `pbjs.setConfig` API](#pbjs.setConfig).

{: .alert.alert-success :}
For a complete list of methods that were removed, see the [Publisher API Reference]({{site.baseurl}}/dev-docs/publisher-api-reference.html).

<a name="pbjs.setConfig" />

## Legacy APIs replaced by `pbjs.setConfig`

For 1.0, the following APIs were removed in favor of a generic "options" param object passed to [`pbjs.setConfig`](/dev-docs/publisher-api-reference/setConfig.html):

- `pbjs.bidderTimeout` - use `pbjs.setConfig({bidderTimeout})` instead
- `pbjs.logging` - use `pbjs.setConfig({debug})` instead
- `pbjs.publisherDomain` - use `pbjs.setConfig({publisherDomain})` instead
- `pbjs.setPriceGranularity` - use `pbjs.setConfig({priceGranularity})` instead
- `pbjs.enableSendAllBids`- use `pbjs.setConfig({enableSendAllBids})` instead. Now defaults to `true`.
- `pbjs.setBidderSequence` - use `pbjs.setConfig({bidderSequence})` instead
- `pbjs.setS2SConfig` - use `pbjs.setConfig({s2sConfig})` instead

### `pbjs.setConfig` Example
{:.no_toc}

{: .alert.alert-warning :}
The input to `pbjs.setConfig` must be JSON (no JavaScript functions are allowed).

{% highlight js %}

pbjs.setConfig({
    currency: {
        "adServerCurrency": "JPY",
        "conversionRateFile": "<url>"
    },
    debug: true, // Previously `logging`
    s2sConfig: { ... },
    priceGranularity: "medium",
    enableSendAllBids: false, // Default will be `true` as of 1.0
    bidderSequence: "random", // Default is random
    bidderTimeout: 700, // Default for all requests.
    publisherDomain: "abc.com", // Used for SafeFrame creative.
    pageOptions: { ... },
    cache: {url: "<prebid cache url>"}
});

{% endhighlight %}

## No More Default Endpoints

In Prebid 0.x there were defaults for the Prebid Server endpoints and the video cache URL. With 1.0, these defaults have been removed to be vendor neutral, so all publisher pages must define them via `pbjs.setConfig()`. The same functionality as 0.x may be achieved as shown below:

{% highlight js %}

pbjs.setConfig({
    cache: {url: "https://prebid.adnxs.com/pbc/v1/cache"},
    s2sConfig: {
        ...
        endpoint: "https://prebid.adnxs.com/pbs/v1/openrtb2/auction",
        syncEndpoint: "https://prebid.adnxs.com/pbs/v1/cookie_sync",
        ...
    }
});

{% endhighlight %}

## Size Mapping Changes

The previous [`sizeMapping` functionality]({{site.baseurl}}/dev-docs/examples/size-mapping.html) was removed and replaced by a `sizeConfig` parameter to the `pbjs.setConfig` method that provides a more powerful way to describe types of devices and
screens.

If `sizeConfig` is passed to `pbjs.setConfig`:

- Before `requestBids` sends bids requests to adapters, it will evaluate and pick the appropriate label(s) based on the `sizeConfig.mediaQuery` and device properties and then filter the `adUnit.bids` based on the `labels` defined (by dropping those ad units that don't match the label definition).

 - The `sizeConfig.mediaQuery` property allows media queries in the form described [here](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries).  They are tested using the [`window.matchMedia` API](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia).

- If a label conditional (e.g. labelAny) doesn???t exist on an ad unit, it is automatically included in all requests for bids.

- If multiple rules match, the sizes will be filtered to the intersection of all matching rules' `sizeConfig.sizesSupported` arrays.

- The `adUnit.sizes` selected will be filtered based on the `sizesSupported` property of the matched `sizeConfig`. So the `adUnit.sizes` is a subset of the sizes defined from the resulting intersection of `sizesSupported` sizes and `adUnit.sizes`.

### `sizeConfig` Example
{:.no_toc}

To set size configuration rules, you can pass `sizeConfig` into `pbjs.setConfig` as follows:

{% highlight js %}

pbjs.setConfig({
  sizeConfig: [{
    'mediaQuery': '(min-width: 1200px)',
    'sizesSupported': [
      [970, 90],
      [728, 90],
      [300, 250]
    ],
    'labels': ['desktop']
  }, {
    'mediaQuery': '(min-width: 768px) and (max-width: 1199px)',
    'sizesSupported': [
      [728, 90],
      [300, 250]
    ],
    'labels': ['tablet', 'phone']
  }, {
    'mediaQuery': '(min-width: 0px)',
    'sizesSupported': [
      [300, 250],
      [300, 100]
    ],
    'labels': ['phone']
  }]
});

{% endhighlight %}

### Labels
{:.no_toc}

Labels can now be specified as a property on either an `adUnit` or on `adUnit.bids[]`.  The presence of a label will disable the adUnit or bidder unless a sizeConfig rule has matched and enabled the label or the label has been enabled manually by passing them into through requestBids: `pbjs.requestBids({labels:[]})`.  

Labels may be targeted in the AdUnit structure by two conditional operators: `labelAny` and `labelAll`.

With the `labelAny` operator, just one label has to match for the condition to be true. In the example below, either A or B can be defined in the label array to activate the bid or ad unit:
{% highlight bash %}
labelAny: ["A", "B"]
{% endhighlight %}

With the `labelAll` conditional, every element of the target array must match an element of the label array in
order for the condition to be true. In the example below, both A and B must be defined in the label array to activate the bid or ad unit:
{% highlight bash %}
labelAll: ["A", "B"]
{% endhighlight %}

{: .alert.alert-warning :}
Only one conditional may be specified on a given AdUnit or bid -- if both `labelAny` and `labelAll` are specified, only the first one will be utilized and an error will be logged to the console. It is allowable for an AdUnit to have one condition and a bid to have another.

{: .alert.alert-warning :}
If either `labelAny` or `labelAll` values is an empty array, it evaluates to `true`.


Defining labels on the adUnit looks like the following:

{% highlight js %}

pbjs.addAdUnits([{
    code: "ad-slot-1",
    mediaTypes: {
        banner: {
            sizes: [
                [970, 90],
                [728, 90],
                [300, 250],
                [300, 100]
            ]
        }
    },
    labelAny: ["visitor-uk"]
    /* The full set of bids, not all of which are relevant on all devices */
    bids: [{
            bidder: "pulsepoint",
            /* Labels flag this bid as relevant only on these screen sizes. */
            labelAny: ["desktop", "tablet"],
            params: {
                "cf": "728X90",
                "cp": 123456,
                "ct": 123456
            }
        },
        {
            bidder: "pulsepoint",
            labelAny: ["desktop", "phone"],
            params: {
                "cf": "300x250",
                "cp": 123456,
                "ct": 123456
            }
        },
        {
            bidder: "sovrn",
            labelAny: ["desktop", "tablet"],
            params: {
                "tagid": "123456"
            }
        },
        {
            bidder: "sovrn",
            labelAny: ["phone"],
            params: {
                "tagid": "111111"
            }
        }
    ]
}]);

{% endhighlight %}

See [Conditional Ad Units]({{site.baseurl}}/dev-docs/conditional-ad-units.html) for additional use cases around labels.


### Manual Label Configuration
{:.no_toc}

If an adUnit and/or adUnit.bids[] bidder has labels defined, they will be disabled by default.  Manually setting active labels through `pbjs.requestBids()` will enable the selected adUnits and/or bidders.

You can manually turn on labels using the following code:

{% highlight js %}

pbjs.requestBids({
  labels: ['visitor-uk']
});

{% endhighlight %}

## Ad Unit Changes

The `mediaType` attribute has been removed in favor of a `mediaTypes` object. This object accepts multiple properties (i.e. `video`, `banner`, `native` etc) with an optional key-value pair object nested inside, e.g.:

{% highlight js %}

adUnit = {
    "code": "unique_code_for_placement"
    "mediaTypes": { // New field to replace `mediaType`. Defaults to `banner` if not specified.
        video: {
            context: 'outstream',
            playerSize: [600, 480]
        },
        banner: {
            sizes: [300, 250],
            ...options
        },
        native: { ...options }
    },
    labelAny: ["desktop", "mobile"]
    bids: { ... } // Same as existing definition with addition of `label` attribute.
}

{% endhighlight %}

## Further Reading

+ [Publisher API Reference]({{site.baseurl}}/dev-docs/publisher-api-reference.html)
