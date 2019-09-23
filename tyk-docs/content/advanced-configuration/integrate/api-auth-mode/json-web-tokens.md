---
date: 2017-03-24T16:40:31Z
title: Integrate with JSON Web Tokens
menu:
  main:
    parent: "API Authentication Mode"
weight: 0
---

## <a name="using-jwts-with-a-3rd-party"></a> Using JSON Web Tokens with a 3rd Party

JSON Web tokens (JWTs) are an excellent, self-contained way to integrate a Tyk Gateway with a third party Identity Provider without needing any direct integration.

With Tyk, so long as a JWT provides two simple claims, only one of which is Tyk specific, integration can be easily achieved.

To integrate a JWT based IDP (Identity Provider) with Tyk, all you will need to do is ensure that your IDP can add a custom claim to the JWT that lists the policy ID to use for the bearer of the token. Tyk will take care of the rest, ensuring that the rate limits and quotas of the underlying identity of the bearer are maintained across JWT token re-issues, so long as the "sub" (or whichever identity claim you chose to use) is available and consistent throughout and the policy that underpins the security clearance of the token exists too.

See [JSON Web Tokens](/docs/basic-config-and-security/security/your-apis/json-web-tokens/) in the **Security** section for more details.