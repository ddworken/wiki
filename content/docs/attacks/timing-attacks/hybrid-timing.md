+++
title = "Hybrid Timing"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "iframes",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP",
]
menu = "main"
weight = 3
+++

Hybrid Timing Attacks allow attackers to measure a combination of factors that influence the final timing measurement. These factors can be:

- [Network]({{< ref "network-timing.md" >}}).
- Parsing.
- Retrieve and load subresources.
- [Execution]({{< ref "execution-timing.md" >}}).

Some of the factors differ in value depending on the application. This means that [Network Timing]({{< ref "network-timing.md" >}}) might be more observable in pages with more backend processing while [Execution Timing]({{< ref "execution-timing.md" >}}) can be more observable in applications processing and displaying data within the browser. Attackers can also eliminate some of the factors of the equation to obtain better measurements, for example, one could preload all the subresources by embedding the page as an `iframe` (forcing the browser to cache them) and do a second measurement which will exclude any delay introduced by the retrieval of those subresources.

##  Frame Timing Attacks (Hybrid)

If a page does not set [Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}}), an attacker can obtain a hybrid measurement that considers all the factors. This attack is similar to the [Network-based Attack]({{< ref "network-timing.md#frame-timing-attacks-network" >}}), but when the resource is retrieved the page is rendered and executed by the browser (subresources fetched and JavaScript executed). In this scenario, the `onload` event only triggers when the page fully loads.

```html
<iframe name=f id=g></iframe>
<script>
before = performance.now();
f.location = 'https://target.page';
g.onerror = g.onload = ()=>{
    console.log('time was', performance.now() - before)
};
</script>
```

## Defense

| Attack Alternative  | [Same-Site Cookies]({{< ref "../../defenses/opt-in/same-site-cookies.md" >}})  | [Fetch Metadata]({{< ref "../../defenses/opt-in/fetch-metadata.md" >}})  | [COOP]({{< ref "../../defenses/opt-in/coop.md" >}})  |  [Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}}) |
|:----------------------:|:------------------:|:---------------:|:-----:|:--------------------:|
| Frame Timing (Hybrid)  |         ✔️       |      ✔️       |  ❌   |          ✔️          |
