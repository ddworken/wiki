+++
title = "Execution Timing"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "Event Loop",
    "Service Workers",
    "Site Isolation",
    "CSS Injections",
    "Regex Injections",
    "iframes",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP",
    "Framing Protections",
]
menu = "main"
weight = 3
+++

Measuring the time of JavaScript execution in a browser can give attackers information on when certain events are triggered, and how long some operations take. 

## Timing the Event Loop

JavaScript concurrency model is based on a [single-threaded event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop) which means it can only run one task at a time. If, for example, some time-consuming task blocks the event loop, the user can perceive a freeze on a page as a result of the UI thread being starved. Other tasks must wait until the blocking one runs to conclusion. Each browser implements different [process models](https://www.chromium.org/developers/design-documents/process-models), which means some web sites might run in different threads (and event loops) depending on their relations.

Some attacks exploit this model to steal secrets from a cross-origin page:

- Infer how long code from a different origin takes to run by measuring how long it takes to run next in the event pool [^1] [^2]. The attacker keeps sending events to the event loop with fixed properties, which will eventually be dispatched if the pool is empty. Other origins will dispatch events to the same pool, and this is where an attacker infers the timing difference by detecting if a delay occurred with one of its tasks.
- Steal a secret from a cross-origin page if the said secret is being compared by an attacker-controlled string. The leak is a result of comparing timing differences in the event loop of a char-by-char string comparison [^2] (using the previous technique). In browsers without [process isolation](https://www.chromium.org/Home/chromium-security/site-isolation), cross-window communications between different origins will run in the same thread, thus sharing the same event loop.

{{< hint warning >}}
This attack is no longer possible in Browsers with process isolation mechanisms in place. Such mechanisms are only present in Chromium-Based browsers with [Site Isolation](https://www.chromium.org/Home/chromium-security/site-isolation) and *soon* in Firefox under [Project Fission](https://wiki.mozilla.org/Project_Fission).
{{< /hint >}}

## Busy Event Loop

Another technique to measure JavaScript Execution consists of blocking the event loop of a thread and time how long does it take for the event loop to be available again. One of the main advantages of this attack is its ability to circumvent Site Isolation as an attacker origin can mess with the execution of another origin. The attack works as follows:

1. Navigate the target website in a separate window with `window.open` or inside an `iframe` (if [Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}}) are **not** in place).
2. Wait for the long computation to start.
3. Load any same-site page inside an `iframe`, regardless of any [Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}}). 

An attacker can detect how long the target website is executed by timing how long it took for the `iframe` (in step 3) to trigger the `onload` event ([Network Timing]({{< ref "network-timing.md" >}}) of step 3 should be despicable). Since both navigations occurred within the same context and they are same-site, they run in the same thread and share the same event loop (they can block each other).

The example below shows how the measurement can be obtained, using the same technique described in [Cross-Window (Network) Timing Attacks]({{< ref "network-timing.md#cross-window-timing-attacks" >}}) for step 2 to detect when the window stated loading.

```javascript
let w = 0, end = 0, begin = 0;
onmessage=()=>{
  try{
    if(w && w.document.cookie){
      // still same origin
    }
    postMessage('','*');
  }catch(e){
    begin = performance.now();
    var x = document.createElement('iframe');
    x.src = "https://any-same-site.target.page";
    document.body.appendChild(x);
    start = performance.now();
    x.onload = () => console.log(performance.now() - begin)
  }
};
postMessage('','*');
w = open('https://target.page');
```

## Service Workers

[Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) can be used to offer offline solutions to web applications but it might be abused by attackers to measure the timing of javascript execution[^4]. They serve as a `proxy` between the browser and the network and allow applications to intercept any network requests made by the main thread (document).

To make a timing measurement an attacker can perform the following steps:

1. The attacker registers a service worker in one of its domains (attacker.com).
2. In the main document, the attacker issues a navigation (window.open) to the target website and instructs the Service Worker to start a timer.
3. When the new window starts loading the attacker will navigate the reference obtained in step 2 to a page handled by the service worker.
4. When the request performed in step 3 arrives to the Service Worker it will return a 204 (No Content) response, which will abort the navigation.
5. At this point the Service Worker will collect a measurement from the timer started in step 2. This measurement will be affected by how long JavaScript blocked the navigation for.

Since the navigation won't actually happen, steps from 3 to 5 can be repeated to get more measurements on successive JavaScript execution timings.

## CSS Injections

{{< hint info >}}
This group of XS-Leaks requires a CSS Injection on the target page.
{{< /hint >}}

Among the different CSS Injection vectors, the most noticeable one is the abuse of CSS Selectors. They can be used as an expression to match and select certain HTML elements. For example, the selector `input[value^="a"]` will be matched if the value of an `input` tag starts with the character "a". So, to detect if a CSS Selector matched the expression, attackers could trigger a callback to one of their websites using certain properties like `background`, `@import`, etc [^6] [^7]. The matching process can be easily brute-forced, and extended to the full string.

### jQuery, CSS Selectors & Short-circuit Timing

Attackers can abuse another interesting behavior of CSS selectors which is `short-circuit` evaluation of expressions. This expression is received in an `URL` hash and evaluated if the page executes `jQuery(location.hash)` [^3].

A timing attack is possible because the expression is compared from right to left, so if the selector `main[id='site-main']` does not match and fails to evaluate, the other parts of the selector (`*:has(*:has(*:has(*))))`) which take longer to execute, are ignored (just like the `and` operator but backwards).

```javascript
$("*:has(*:has(*:has(*)) *:has(*:has(*:has(*))) *:has(*:has(*:has(*)))) main[id='site-main']")
```

{{< hint good >}}
This attack is no longer possible in Browsers with process isolation mechanisms in place. Such mechanisms are only present in Chromium-Based browsers with [Site Isolation](https://www.chromium.org/Home/chromium-security/site-isolation) and *soon* in Firefox under [Project Fission](https://wiki.mozilla.org/Project_Fission).
{{< /hint >}}

{{< hint good >}}
In browsers with process isolation mechanisms, [Service Workers]({{< ref "execution-timing.md#service-workers" >}}) can be abused to obtain the execution timing measurement or tricks like [Busy Event Loop tricks]({{< ref "#busy-event-loop" >}}) to circumvent Site Isolation.
{{< /hint >}}

## ReDoS

{{< hint info >}}
This group of XS-Leaks requires an injection of Regex Expressions on the target page.
{{< /hint >}}

Regular Expression Denial of Service (ReDoS) it's an attack which result in a Denial of Service in applications that allow Regex as user input [^2] [^5]. The DoS results from an injected Regex that would run in exponential time. Some attacks applied this principle into leaking information: The attacker's injection cause a DoS if the Regex matches a character in some secret and computes quickly otherwise. This could happen in both client and server side.


## Defense

| Attack Alternative  | [Same-Site Cookies]({{< ref "../../defenses/opt-in/same-site-cookies.md" >}})  | [Fetch Metadata]({{< ref "../../defenses/opt-in/fetch-metadata.md" >}})  | [COOP]({{< ref "../../defenses/opt-in/coop.md" >}})  |  [Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}}) |
|:-------------------:|:------------------:|:---------------:|:-----:|:--------------------:|
| T. Event Loop       |         ✔️  [(if Strict)]({{< ref "../../defenses/opt-in/same-site-cookies.md#lax-vs-strict" >}})        |      ✔️         |  ❓   |          ❌         |
| Service Workers     |         ✔️         |      ✔️         |  ✔️   |          ❌         |
| jQuery              |         ✔️         |      ✔️         |  ❌   |          ❌         |
| ReDoS               |         ✔️         |      ✔️         |  ❌   |          ❌         |
| Busy Event Loop     |         ✔️         |      ✔️         |  ❌   |          ❌         |

## References

[^1]: Loophole: Timing Attacks on Shared Event Loops in Chrome, [link](https://www.usenix.org/system/files/conference/usenixsecurity17/sec17-vila.pdf)
[^2]: Matryoshka - Web Application Timing Attacks (or.. Timing Attacks against JavaScript Applications in Browsers), [link](https://sirdarckcat.blogspot.com/2014/05/matryoshka-web-application-timing.html)
[^3]: A timing attack with CSS selectors and Javascript, [link](https://blog.sheddow.xyz/css-timing-attack/)
[^4]: Security: XS-Search + XSS Auditor = Not Cool, [link](https://bugs.chromium.org/p/chromium/issues/detail?id=922829)
[^5]: A Rough Idea of Blind Regular Expression Injection Attack, [link](https://diary.shift-js.info/blind-regular-expression-injection/)
[^6]: CSS Injection Primitives, [link](https://x-c3ll.github.io/posts/CSS-Injection-Primitives/)
[^7]: HTTPLeaks, [link](https://github.com/cure53/HTTPLeaks/)