+++
title = "CORP Leaks"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "Browser Feature",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
]
menu = "main"
weight = 2
+++

## Explanation

[Cross-Origin Resource Policy]({{< ref "../../defenses/opt-in/corp.md" >}}) (CORP) is a web platform security feature which enforces [CORB]({{< ref "../../defenses/browser-default/corb.md" >}}). Unfortunately, similarly to [CORB XS-Leak]({{< ref "corb.md" >}}) applications can introduce a new XS-Leak if they misconfigure the use of this protection.

A webpage will introduce an XS-Leak if `CORP` is enforced based on user data. If a page search feature enforces `CORP` when showing results, but doesn't when returning no results, an attacker will be able to distinguish the two scenarios. This occurs because a page/resource protected by `CORP` will return a network error when fetched.

## Defense

An application can avoid this XS-Leak if it guarantees `CORP` is deployed in all application resources/endpoints. Moreover generic security mechanisms that allow the invalidation of cross-site requests will also help preventing this attack.

| [Same-Site Cookies]({{< ref "../../defenses/opt-in/same-site-cookies.md" >}})  | [Fetch Metadata]({{< ref "../../defenses/opt-in/fetch-metadata.md" >}})  | [COOP]({{< ref "../../defenses/opt-in/coop.md" >}})  |  [Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}}) |
|:------------------:|:---------------:|:-----:|:--------------------:|
|         ✔️         |      ✔️         |  ❌   |          ❌         |
