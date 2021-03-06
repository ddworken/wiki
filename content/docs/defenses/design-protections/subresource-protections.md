+++
title = "Subresource Protections"
description = "Subresources Protections"
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

## Random tokens

One of the principles of protecting subresources is the same as protecting endpoints from [CSRF attacks](https://owasp.org/www-community/attacks/csrf). The difference from [CSRF protections](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) is that in the case of XS-Leaks, `GET` requests are the ones usually worth protecting. To apply this protection applications can append a (cryptographically strong) pseudorandom value, unique to each request/session, to make the URL of a subresource unpredictable to an attacker. The protection can be applied to the following types of subresources:

- Authenticated subresources such as API endpoints or regular authenticated URLs. While pseudorandom values can be used in this case, security mitigations like [Same-Site Cookies]({{< ref "../opt-in/same-site-cookies.md" >}}) can be cheaper to deploy and more effective.
- Unauthenticated subresources such as images can use this protection to prevent some types of [Cache Probing Attacks]({{< ref "../../attacks/cache-probing.md" >}}). In this scenario, this protection can be highly effective but it might hinder cache efficacy or be otherwise hard to deploy.

## User Consent

Some applications might ask for user consent to trigger a certain sensitive action. Facebook deploys this protection in some sensible search endpoints like `https://www.facebook.com/messages/?qa=UserMustConsent`, where a user musk press OK to advance with the search query. Since attackers can't surpass this verification, the page won't leak any special behavior.

User consent is often asked in applications to warn the user it's being redirected to a website **outside** of the current website. This prevents attackers to [detect some type of navigations]({{< ref "../../attacks/navigations.md" >}}).

## Deployment

While this protection might work in some scenarios, it has some disadvantages:

- Hard to deploy as it requires substantial changes in the codebase. 
- It might break the desired behavior for the feature.
- In the case of random tokens, it will break bookmarks and other permanent references.
- Consent pages might add friction to using the application.

{{< hint warning >}}
This protection can be enough to fix attacks temporarily in certain scenarios. Due to the challenges of deploying this protection, applications are encouraged to deploy [opt-in web platform security features]({{< ref "../_index.md" >}}) as the default approach.
{{< /hint >}}
