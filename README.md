# .github
    "instanceUrl": "https://devdiv.visualstudio.com/",
    "template": "TFSDEVDIV",
    "projectName": "DEVDIV",
    "areaPath": "DevDiv\\NET Fundamentals\\.NET Acquisition\\Docker",
    "iterationPath": "DevDiv",
    "notificationAliases": [ "dotnetADTeam@microsoft.com" ],
    "repositoryName"illumex",
    "codebaseName": "github"
WHATWG
HTML
Living Standard — Last Updated 7 March 2024

← 6.12 The popover attribute — Table of Contents — 7.2 APIs related to navigation and session history →
7 Loading web pages
7.1 Supporting concepts
7.1.1 Origins
7.1.1.1 Sites
7.1.1.2 Relaxing the same-origin restriction
7.1.2 Origin-keyed agent clusters
7.1.3 Cross-origin opener policies
7.1.3.1 The headers
7.1.3.2 Browsing context group switches due to cross-origin opener policy
7.1.3.3 Reporting
7.1.4 Cross-origin embedder policies
7.1.4.1 The headers
7.1.4.2 Embedder policy checks
7.1.5 Sandboxing
7.1.6 Policy containers
7 Loading web pages
This section describes features that apply most directly to web browsers. Having said that, except where specified otherwise, the requirements defined in this section do apply to all user agents, whether they are web browsers or not.

7.1 Supporting concepts
7.1.1 Origins
Origins are the fundamental currency of the web's security model. Two actors in the web platform that share an origin are assumed to trust each other and to have the same authority. Actors with differing origins are considered potentially hostile versus each other, and are isolated from each other to varying degrees.

For example, if Example Bank's web site, hosted at bank.example.com, tries to examine the DOM of Example Charity's web site, hosted at charity.example.org, a "SecurityError" DOMException will be raised.

An origin is one of the following:

An opaque origin
An internal value, with no serialization it can be recreated from (it is serialized as "null" per serialization of an origin), for which the only meaningful operation is testing for equality.

A tuple origin
A tuple consists of:

A scheme (an ASCII string).
A host (a host).
A port (null or a 16-bit unsigned integer).
A domain (null or a domain). Null unless stated otherwise.
Origins can be shared, e.g., among multiple Document objects. Furthermore, origins are generally immutable. Only the domain of a tuple origin can be changed, and only through the document.domain API.

The effective domain of an origin origin is computed as follows:

If origin is an opaque origin, then return null.

If origin's domain is non-null, then return origin's domain.

Return origin's host.

The serialization of an origin is the string obtained by applying the following algorithm to the given origin origin:

If origin is an opaque origin, then return "null".

Otherwise, let result be origin's scheme.

Append "://" to result.

Append origin's host, serialized, to result.

If origin's port is non-null, append a U+003A COLON character (:), and origin's port, serialized, to result.

Return result.

The serialization of ("https", "xn--maraa-rta.example", null, null) is "https://xn--maraa-rta.example".

There used to also be a Unicode serialization of an origin. However, it was never widely adopted.

Two origins, A and B, are said to be same origin if the following algorithm returns true:

If A and B are the same opaque origin, then return true.

If A and B are both tuple origins and their schemes, hosts, and port are identical, then return true.

Return false.

Two origins, A and B, are said to be same origin-domain if the following algorithm returns true:

If A and B are the same opaque origin, then return true.

If A and B are both tuple origins, run these substeps:

If A and B's schemes are identical, and their domains are identical and non-null, then return true.

Otherwise, if A and B are same origin and their domains are identical and null, then return true.

Return false.

A	B	same origin	same origin-domain
("https", "example.org", null, null)	("https", "example.org", null, null)	✅	✅
("https", "example.org", 314, null)	("https", "example.org", 420, null)	❌	❌
("https", "example.org", 314, "example.org")	("https", "example.org", 420, "example.org")	❌	✅
("https", "example.org", null, null)	("https", "example.org", null, "example.org")	✅	❌
("https", "example.org", null, "example.org")	("http", "example.org", null, "example.org")	❌	❌
7.1.1.1 Sites
A scheme-and-host is a tuple of a scheme (an ASCII string) and a host (a host).

A site is an opaque origin or a scheme-and-host.

To obtain a site, given an origin origin, run these steps:

If origin is an opaque origin, then return origin.

If origin's host's registrable domain is null, then return (origin's scheme, origin's host).

Return (origin's scheme, origin's host's registrable domain).

Two sites, A and B, are said to be same site if the following algorithm returns true:

If A and B are the same opaque origin, the return true.

If A or B is an opaque origin, then return false.

If A's and B's scheme values are different, then return false.

If A's and B's host values are not equal, then return false.

Return true.

The serialization of a site is the string obtained by applying the following algorithm to the given site site:

If site is an opaque origin, then return "null".

Let result be site[0].

Append "://" to result.

Append site[1], serialized, to result.

Return result.

It needs to be clear from context that the serialized value is a site, not an origin, as there is not necessarily a syntactic difference between the two. For example, the origin ("https", "shop.example", null, null) and the site ("https", "shop.example") have the same serialization: "https://shop.example".

Two origins, A and B, are said to be schemelessly same site if the following algorithm returns true:

If A and B are the same opaque origin, then return true.

If A and B are both tuple origins, then:

Let hostA be A's host, and let hostB be B's host.

If hostA equals hostB and hostA's registrable domain is null, then return true.

If hostA's registrable domain equals hostB's registrable domain and is non-null, then return true.

Return false.

Two origins, A and B, are said to be same site if the following algorithm returns true:

Let siteA be the result of obtaining a site given A.

Let siteB be the result of obtaining a site given B.

If siteA is same site with siteB, then return true.

Return false.

Unlike the same origin and same origin-domain concepts, for schemelessly same site and same site, the port and domain components are ignored.

For the reasons explained in URL, the same site and schemelessly same site concepts should be avoided when possible, in favor of same origin checks.

Given that wildlife.museum, museum, and com are public suffixes and that example.com is not:

A	B	schemelessly same site	same site
("https", "example.com")	("https", "sub.example.com")	✅	✅
("https", "example.com")	("https", "sub.other.example.com")	✅	✅
("https", "example.com")	("http", "non-secure.example.com")	✅	❌
("https", "r.wildlife.museum")	("https", "sub.r.wildlife.museum")	✅	✅
("https", "r.wildlife.museum")	("https", "sub.other.r.wildlife.museum")	✅	✅
("https", "r.wildlife.museum")	("https", "other.wildlife.museum")	❌	❌
("https", "r.wildlife.museum")	("https", "wildlife.museum")	❌	❌
("https", "wildlife.museum")	("https", "wildlife.museum")	✅	✅
("https", "example.com")	("https", "example.com.")	❌	❌
(Here we have omitted the port and domain components since they are not considered.)

7.1.1.2 Relaxing the same-origin restriction
document.domain [ = domain ]
Returns the current domain used for security checks.

Can be set to a value that removes subdomains, to change the origin's domain to allow pages on other subdomains of the same domain (if they do the same thing) to access each other. This enables pages on different hosts of a domain to synchronously access each other's DOMs.

In sandboxed iframes, Documents with opaque origins, and Documents without a browsing context, the setter will throw a "SecurityError" exception. In cases where crossOriginIsolated or originAgentCluster return true, the setter will do nothing.

Avoid using the document.domain setter. It undermines the security protections provided by the same-origin policy. This is especially acute when using shared hosting; for example, if an untrusted third party is able to host an HTTP server at the same IP address but on a different port, then the same-origin protection that normally protects two different sites on the same host will fail, as the ports are ignored when comparing origins after the document.domain setter has been used.

Because of these security pitfalls, this feature is in the process of being removed from the web platform. (This is a long process that takes many years.)

Instead, use postMessage() or MessageChannel objects to communicate across origins in a safe manner.

The domain getter steps are:

Let effectiveDomain be this's origin's effective domain.

If effectiveDomain is null, then return the empty string.

Return effectiveDomain, serialized.

The domain setter steps are:

If this's browsing context is null, then throw a "SecurityError" DOMException.

If this's active sandboxing flag set has its sandboxed document.domain browsing context flag set, then throw a "SecurityError" DOMException.

Let effectiveDomain be this's origin's effective domain.

If effectiveDomain is null, then throw a "SecurityError" DOMException.

If the given value is not a registrable domain suffix of and is not equal to effectiveDomain, then throw a "SecurityError" DOMException.

If the surrounding agent's agent cluster's is origin-keyed is true, then return.

Set this's origin's domain to the result of parsing the given value.

To determine if a scalar value string hostSuffixString is a registrable domain suffix of or is equal to a host originalHost:

If hostSuffixString is the empty string, then return false.

Let hostSuffix be the result of parsing hostSuffixString.

If hostSuffix is failure, then return false.

If hostSuffix does not equal originalHost, then:

If hostSuffix or originalHost is not a domain, then return false.

This excludes hosts that are IP addresses.

If hostSuffix, prefixed by U+002E (.), does not match the end of originalHost, then return false.

If any of the following are true:

hostSuffix equals hostSuffix's public suffix; or

hostSuffix, prefixed by U+002E (.), matches the end originalHost's public suffix,

then return false. [URL]

Assert: originalHost's public suffix, prefixed by U+002E (.), matches the end of hostSuffix.

Return true.

hostSuffixString	originalHost	Outcome of is a registrable domain suffix of or is equal to	Notes
"0.0.0.0"	0.0.0.0	✅	
"0x10203"	0.1.2.3	✅	
"[0::1]"	::1	✅	
"example.com"	example.com	✅	
"example.com"	example.com.	❌	Trailing dot is significant.
"example.com."	example.com	❌
"example.com"	www.example.com	✅	
"com"	example.com	❌	At the time of writing, com is a public suffix.
"example"	example	✅	
"compute.amazonaws.com"	example.compute.amazonaws.com	❌	At the time of writing, *.compute.amazonaws.com is a public suffix.
"example.compute.amazonaws.com"	www.example.compute.amazonaws.com	❌
"amazonaws.com"	www.example.compute.amazonaws.com	❌
"amazonaws.com"	test.amazonaws.com	✅	At the time of writing, amazonaws.com is a registrable domain.
7.1.2 Origin-keyed agent clusters
window.originAgentCluster
Returns true if this Window belongs to an agent cluster which is origin-keyed, in the manner described in this section.

A Document delivered over a secure context can request that it be placed in an origin-keyed agent cluster, by using the `Origin-Agent-Cluster` HTTP response header. This header is a structured header whose value must be a boolean. [STRUCTURED-FIELDS]

Per the processing model in the create and initialize a new Document object, values that are not the structured header boolean true value (i.e., `?1`) will be ignored.

The consequences of using this header are that the resulting Document's agent cluster key is its origin, instead of the corresponding site. In terms of observable effects, this means that attempting to relax the same-origin restriction using document.domain will instead do nothing, and it will not be possible to send WebAssembly.Module objects to cross-origin Documents (even if they are same site). Behind the scenes, this isolation can allow user agents to allocate implementation-specific resources corresponding to agent clusters, such as processes or threads, more efficiently.

Note that within a browsing context group, the `Origin-Agent-Cluster` header can never cause same-origin Document objects to end up in different agent clusters, even if one sends the header and the other doesn't. This is prevented by means of the historical agent cluster key map.

This means that the originAgentCluster getter can return false, even if the header is set, if the header was omitted on a previously-loaded same-origin page in the same browsing context group. Similarly, it can return true even when the header is not set.

The originAgentCluster getter steps are to return the surrounding agent's agent cluster's is origin-keyed.

Documents with an opaque origin can be considered unconditionally origin-keyed; for them the header has no effect, and the originAgentCluster getter will always return true.

Similarly, Documents whose agent cluster's cross-origin isolation mode is not "none" are automatically origin-keyed. The `Origin-Agent-Cluster` header might be useful as an additional hint to implementations about resource allocation, since the `Cross-Origin-Opener-Policy` and `Cross-Origin-Embedder-Policy` headers used to achieve cross-origin isolation are more about ensuring that everything in the same address space opts in to being there. But adding it would have no additional observable effects on author code.

7.1.3 Cross-origin opener policies
A cross-origin opener policy value allows a document which is navigated to in a top-level browsing context to force the creation of a new top-level browsing context, and a corresponding group. The possible values are:

"unsafe-none"
This is the (current) default and means that the document will occupy the same top-level browsing context as its predecessor, unless that document specified a different cross-origin opener policy.

"same-origin-allow-popups"
This forces the creation of a new top-level browsing context for the document, unless its predecessor specified the same cross-origin opener policy and they are same origin.

"same-origin"
This behaves the same as "same-origin-allow-popups", with the addition that any auxiliary browsing context created needs to contain same origin documents that also have the same cross-origin opener policy or it will appear closed to the opener.

"same-origin-plus-COEP"
This behaves the same as "same-origin", with the addition that it sets the (new) top-level browsing context's group's cross-origin isolation mode to one of "logical" or "concrete".

"same-origin-plus-COEP" cannot be directly set via the `Cross-Origin-Opener-Policy` header, but results from a combination of setting both `Cross-Origin-Opener-Policy: same-origin` and a `Cross-Origin-Embedder-Policy` header whose value is compatible with cross-origin isolation together.

A cross-origin opener policy consists of:

A value, which is a cross-origin opener policy value, initially "unsafe-none".

A reporting endpoint, which is string or null, initially null.

A report-only value, which is a cross-origin opener policy value, initially "unsafe-none".

A report-only reporting endpoint, which is a string or null, initially null.

To match cross-origin opener policy values, given a cross-origin opener policy value A, an origin originA, a cross-origin opener policy value B, and an origin originB:

If A is "unsafe-none" and B is "unsafe-none", then return true.

If A is "unsafe-none" or B is "unsafe-none", then return false.

If A is B and originA is same origin with originB, then return true.

Return false.

7.1.3.1 The headers
✔MDN
A Document's cross-origin opener policy is derived from the `Cross-Origin-Opener-Policy` and `Cross-Origin-Opener-Policy-Report-Only` HTTP response headers. These headers are structured headers whose value must be a token. [STRUCTURED-FIELDS]

The valid token values are the opener policy values. The token may also have attached parameters; of these, the "report-to" parameter can have a valid URL string identifying an appropriate reporting endpoint. [REPORTING]

Per the processing model described below, user agents will ignore this header if it contains an invalid value. Likewise, user agents will ignore this header if the value cannot be parsed as a token.

To obtain a cross-origin opener policy given a response response and an environment reservedEnvironment:

Let policy be a new cross-origin opener policy.

If reservedEnvironment is a non-secure context, then return policy.

Let parsedItem be the result of getting a structured field value given `Cross-Origin-Opener-Policy` and "item" from response's header list.

If parsedItem is not null, then:

If parsedItem[0] is "same-origin", then:

Let coep be the result of obtaining a cross-origin embedder policy from response and reservedEnvironment.

If coep's value is compatible with cross-origin isolation, then set policy's value to "same-origin-plus-COEP".

Otherwise, set policy's value to "same-origin".

If parsedItem[0] is "same-origin-allow-popups", then set policy's value to "same-origin-allow-popups".

If parsedItem[1]["report-to"] exists and it is a string, then set policy's reporting endpoint to parsedItem[1]["report-to"].

Set parsedItem to the result of getting a structured field value given `Cross-Origin-Opener-Policy-Report-Only` and "item" from response's header list.

If parsedItem is not null, then:

If parsedItem[0] is "same-origin", then:

Let coep be the result of obtaining a cross-origin embedder policy from response and reservedEnvironment.

If coep's value is compatible with cross-origin isolation or coep's report-only value is compatible with cross-origin isolation, then set policy's report-only value to "same-origin-plus-COEP".

Report only COOP also considers report-only COEP to assign the special "same-origin-plus-COEP" value. This allows developers more freedom in the order of deployment of COOP and COEP.

Otherwise, set policy's report-only value to "same-origin".

If parsedItem[0] is "same-origin-allow-popups", then set policy's report-only value to "same-origin-allow-popups".

If parsedItem[1]["report-to"] exists and it is a string, then set policy's report-only reporting endpoint to parsedItem[1]["report-to"].

Return policy.

7.1.3.2 Browsing context group switches due to cross-origin opener policy
To check if COOP values require a browsing context group switch, given a boolean isInitialAboutBlank, two origins responseOrigin and activeDocumentNavigationOrigin, and two cross-origin opener policy values responseCOOPValue and activeDocumentCOOPValue:

If the result of matching activeDocumentCOOPValue, activeDocumentNavigationOrigin, responseCOOPValue, and responseOrigin is true, return false.

If all of the following are true:

isInitialAboutBlank;

activeDocumentCOOPValue's value is "same-origin-allow-popups"; and

responseCOOPValue is "unsafe-none",

then return false.

Return true.

To check if enforcing report-only COOP would require a browsing context group switch, given a boolean isInitialAboutBlank, two origins responseOrigin, activeDocumentNavigationOrigin, and two cross-origin opener policies responseCOOP and activeDocumentCOOP:

If the result of checking if COOP values require a browsing context group switch given isInitialAboutBlank, responseOrigin, activeDocumentNavigationOrigin, responseCOOP's report-only value and activeDocumentCOOPReportOnly's report-only value is false, then return false.

Matching report-only policies allows a website to specify the same report-only cross-origin opener policy on all its pages and not receive violation reports for navigations between these pages.

If the result of checking if COOP values require a browsing context group switch given isInitialAboutBlank, responseOrigin, activeDocumentNavigationOrigin, responseCOOP's value and activeDocumentCOOPReportOnly's report-only value is true, then return true.

If the result of checking if COOP values require a browsing context group switch given isInitialAboutBlank, responseOrigin, activeDocumentNavigationOrigin, responseCOOP's report-only value and activeDocumentCOOPReportOnly's value is true, then return true.

Return false.

A cross-origin opener policy enforcement result is a struct with the following items:

A boolean needs a browsing context group switch, initially false.

A boolean would need a browsing context group switch due to report-only, initially false.

A URL url.

An origin origin.

A cross-origin opener policy cross-origin opener policy.

A boolean current context is navigation source, initially false.

To enforce a response's cross-origin opener policy, given a browsing context browsingContext, a URL responseURL, an origin responseOrigin, a cross-origin opener policy responseCOOP, a cross-origin opener policy enforcement result currentCOOPEnforcementResult, and a referrer referrer:

Let newCOOPEnforcementResult be a new cross-origin opener policy enforcement result with

needs a browsing context group switch
currentCOOPEnforcementResult's needs a browsing context group switch
would need a browsing context group switch due to report-only
currentCOOPEnforcementResult's would need a browsing context group switch due to report-only
url
responseURL
origin
responseOrigin
cross-origin opener policy
responseCOOP
current context is navigation source
true
Let isInitialAboutBlank be browsingContext's active document's is initial about:blank.

If isInitialAboutBlank is true and browsingContext's initial URL is null, set browsingContext's initial URL to responseURL.

If the result of checking if COOP values require a browsing context group switch given isInitialAboutBlank, currentCOOPEnforcementResult's cross-origin opener policy's value, currentCOOPEnforcementResult's origin, responseCOOP's value, and responseOrigin is true, then:

Set newCOOPEnforcementResult's needs a browsing context group switch to true.

If browsingContext's group's browsing context set's size is greater than 1, then:

Queue a violation report for browsing context group switch when navigating to a COOP response with responseCOOP, "enforce", responseURL, currentCOOPEnforcementResult's url, currentCOOPEnforcementResult's origin, responseOrigin, and referrer.

Queue a violation report for browsing context group switch when navigating away from a COOP response with currentCOOPEnforcementResult's cross-origin opener policy, "enforce", currentCOOPEnforcementResult's url, responseURL, currentCOOPEnforcementResult's origin, responseOrigin, and currentCOOPEnforcementResult's current context is navigation source.

If the result of checking if enforcing report-only COOP would require a browsing context group switch given isInitialAboutBlank, responseOrigin, currentCOOPEnforcementResult's origin, responseCOOP, and currentCOOPEnforcementResult's cross-origin opener policy, is true, then:

Set result's would need a browsing context group switch due to report-only to true.

If browsingContext's group's browsing context set's size is greater than 1, then:

Queue a violation report for browsing context group switch when navigating to a COOP response with responseCOOP, "reporting", responseURL, currentCOOPEnforcementResult's url, currentCOOPEnforcementResult's origin, responseOrigin, and referrer.

Queue a violation report for browsing context group switch when navigating away from a COOP response with currentCOOPEnforcementResult's cross-origin opener policy, "reporting", currentCOOPEnforcementResult's url, responseURL, currentCOOPEnforcementResult's origin, responseOrigin, and currentCOOPEnforcementResult's current context is navigation source.

Return newCOOPEnforcementResult.

To obtain a browsing context to use for a navigation response, given a browsing context browsingContext, a sandboxing flag set sandboxFlags, a cross-origin opener policy navigationCOOP, and a cross-origin opener policy enforcement result coopEnforcementResult:

If browsingContext is not a top-level browsing context, then return browsingContext.

If coopEnforcementResult's needs a browsing context group switch is false, then:

If coopEnforcementResult's would need a browsing context group switch due to report-only is true, set browsing context's virtual browsing context group ID to a new unique identifier.

Return browsingContext.

Let newBrowsingContext be the first return value of creating a new top-level browsing context and document.

In this case we are going to perform a browsing context group swap. browsingContext will not be used by the new Document that we are about to create. If it is not used by other Documents either (such as ones in the back/forward cache), then the user agent might destroy it at this point.

If navigationCOOP's value is "same-origin-plus-COEP", then set newBrowsingContext's group's cross-origin isolation mode to either "logical" or "concrete". The choice of which is implementation-defined.

It is difficult on some platforms to provide the security properties required by the cross-origin isolated capability. "concrete" grants access to it and "logical" does not.

If sandboxFlags is not empty, then:

Assert navigationCOOP's value is "unsafe-none".

Assert: newBrowsingContext's popup sandboxing flag set is empty.

Set newBrowsingContext's popup sandboxing flag set to a clone of sandboxFlags.

Return newBrowsingContext.

7.1.3.3 Reporting
An accessor-accessed relationship is an enum that describes the relationship between two browsing contexts between which an access happened. It can take the following values:

accessor is opener
The accessor browsing context or one of its ancestors is the opener browsing context of the accessed browsing context's top-level browsing context.

accessor is openee
The accessed browsing context or one of its ancestors is the opener browsing context of the accessor browsing context's top-level browsing context.

none
There is no opener relationship between the accessor browsing context, the accessor browsing context, or any of their ancestors.

To check if an access between two browsing contexts should be reported, given two browsing contexts accessor and accessed, a JavaScript property name P, and an environment settings object environment:

If P is not a cross-origin accessible window property name, then return.

Assert: accessor's active document and accessed's active document are both fully active.

Let accessorTopDocument be accessor's top-level browsing context's active document.

Let accessorInclusiveAncestorOrigins be the list obtained by taking the origin of the active document of each of accessor's active document's inclusive ancestor navigables.

Let accessedTopDocument be accessed's top-level browsing context's active document.

Let accessedInclusiveAncestorOrigins be the list obtained by taking the origin of the active document of each of accessed's active document's inclusive ancestor navigables.

If any of accessorInclusiveAncestorOrigins are not same origin with accessorTopDocument's origin, or if any of accessedInclusiveAncestorOrigins are not same origin with accessedTopDocument's origin, then return.

This avoids leaking information about cross-origin iframes to a top level frame with cross-origin opener policy reporting.

If accessor's top-level browsing context's virtual browsing context group ID is accessed's top-level browsing context's virtual browsing context group ID, then return.

Let accessorAccessedRelationship be a new accessor-accessed relationship with value none.

If accessed's top-level browsing context's opener browsing context is accessor or is an ancestor of accessor, then set accessorAccessedRelationship to accessor is opener.

If accessor's top-level browsing context's opener browsing context is accessed or is an ancestor of accessed, then set accessorAccessedRelationship to accessor is openee.

Queue violation reports for accesses, given accessorAccessedRelationship, accessorTopDocument's cross-origin opener policy, accessedTopDocument's cross-origin opener policy, accessor's active document's URL, accessed's active document's URL, accessor's top-level browsing context's initial URL, accessed's top-level browsing context's initial URL, accessor's active document's origin, accessed's active document's origin, accessor's top-level browsing context's opener origin at creation, accessed's top-level browsing context's opener origin at creation, accessorTopDocument's referrer, accessedTopDocument's referrer, P, and environment.

To sanitize a URL to send in a report given a URL url:

Let sanitizedURL be a copy of url.

Set the username given sanitizedURL and the empty string.

Set the password given sanitizedURL and the empty string.

Return the serialization of sanitizedURL with exclude fragment set to true.

To queue a violation report for browsing context group switch when navigating to a COOP response given a cross-origin opener policy coop, a string disposition, a URL coopURL, a URL previousResponseURL, two origins coopOrigin and previousResponseOrigin, and a referrer referrer:

If coop's reporting endpoint is null, return.

Let coopValue be coop's value.

If disposition is "reporting", then set coopValue to coop's report-only value.

Let serializedReferrer be an empty string.

If referrer is a URL, set serializedReferrer to the serialization of referrer.

Let body be a new object containing the following properties:

key	value
disposition	disposition
effectivePolicy	coopValue
previousResponseURL	If coopOrigin and previousResponseOrigin are same origin this is the sanitization of previousResponseURL, null otherwise.
referrer	serializedReferrer
type	"navigation-to-response"
Queue body as "coop" for coop's reporting endpoint with coopURL.

To queue a violation report for browsing context group switch when navigating away from a COOP response given a cross-origin opener policy coop, a string disposition, a URL coopURL, a URL nextResponseURL, two origins coopOrigin and nextResponseOrigin, and a boolean isCOOPResponseNavigationSource:

If coop's reporting endpoint is null, return.

Let coopValue be coop's value.

If disposition is "reporting", then set coopValue to coop's report-only value.

Let body be a new object containing the following properties:

key	value
disposition	disposition
effectivePolicy	coopValue
nextResponseURL	If coopOrigin and nextResponseOrigin are same origin or isCOOPResponseNavigationSource is true, this is the sanitization of previousResponseURL, null otherwise.
type	"navigation-from-response"
Queue body as "coop" for coop's reporting endpoint with coopURL.

To queue violation reports for accesses, given an accessor-accessed relationship accessorAccessedRelationship, two cross-origin opener policies accessorCOOP and accessedCOOP, four URLs accessorURL, accessedURL, accessorInitialURL, accessedInitialURL, four origins accessorOrigin, accessedOrigin, accessorCreatorOrigin and accessedCreatorOrigin, two referrers accessorReferrer and accessedReferrer, a string propertyName, and an environment settings object environment:

If coop's reporting endpoint is null, return.

Let coopValue be coop's value.

If disposition is "reporting", then set coopValue to coop's report-only value.

If accessorAccessedRelationship is accessor is opener:

Queue a violation report for access to an opened window, given accessorCOOP, accessorURL, accessedURL, accessedInitialURL, accessorOrigin, accessedOrigin, accessedCreatorOrigin, propertyName, and environment.

Queue a violation report for access from the opener, given accessedCOOP, accessedURL, accessorURL, accessedOrigin, accessorOrigin, propertyName, and accessedReferrer.

Otherwise, if accessorAccessedRelationship is accessor is openee:

Queue a violation report for access to the opener, given accessorCOOP, accessorURL, accessedURL, accessorOrigin, accessedOrigin, propertyName, accessorReferrer, and environment.

Queue a violation report for access from an opened window, given accessedCOOP, accessedURL, accessorURL, accessorInitialURL, accessedOrigin, accessorOrigin, accessorCreatorOrigin, and propertyName.

Otherwise:

Queue a violation report for access to another window, given accessorCOOP, accessorURL, accessedURL, accessorOrigin, accessedOrigin, propertyName, and environment

Queue a violation report for access from another window, given accessedCOOP, accessedURL, accessorURL, accessedOrigin, accessorOrigin, and propertyName.

To queue a violation report for access to the opener, given a cross-origin opener policy coop, two URLs coopURL and openerURL, two origins coopOrigin and openerOrigin, a string propertyName, a referrer referrer, and an environment settings object environment:

Let sourceFile, lineNumber and columnNumber be the relevant script URL and problematic position which triggered this report.

Let serializedReferrer be an empty string.

If referrer is a URL, set serializedReferrer to the serialization of referrer.

Let body be a new object containing the following properties:

key	value
disposition	"reporting"
effectivePolicy	coop's report-only value
property	propertyName
openerURL	If coopOrigin and openerOrigin are same origin, this is the sanitization of openerURL, null otherwise.
referrer	serializedReferrer
sourceFile	sourceFile
lineNumber	lineNumber
columnNumber	columnNumber
type	"access-to-opener"
Queue body as "coop" for coop's reporting endpoint with coopURL and environment.

To queue a violation report for access to an opened window, given a cross-origin opener policy coop, three URLs coopURL, openedWindowURL and initialWindowURL, three origins coopOrigin, openedWindowOrigin, and openerInitialOrigin, a string propertyName, and an environment settings object environment:

Let sourceFile, lineNumber and columnNumber be the relevant script URL and problematic position which triggered this report.

Let body be a new object containing the following properties:

key	value
disposition	"reporting"
effectivePolicy	coop's report-only value
property	propertyName
openedWindowURL	If coopOrigin and openedWindowOrigin are same origin, this is the sanitization of openedWindowURL, null otherwise.
openedWindowInitialURL	If coopOrigin and openerInitialOrigin are same origin, this is the sanitization of initialWindowURL, null otherwise.
sourceFile	sourceFile
lineNumber	lineNumber
columnNumber	columnNumber
type	"access-to-opener"
Queue body as "coop" for coop's reporting endpoint with coopURL and environment.

To queue a violation report for access to another window, given a cross-origin opener policy coop, two URLs coopURL and otherURL, two origins coopOrigin and otherOrigin, a string propertyName, and an environment settings object environment:

Let sourceFile, lineNumber and columnNumber be the relevant script URL and problematic position which triggered this report.

Let body be a new object containing the following properties:

key	value
disposition	"reporting"
effectivePolicy	coop's report-only value
property	propertyName
otherURL	If coopOrigin and otherOrigin are same origin, this is the sanitization of otherURL, null otherwise.
sourceFile	sourceFile
lineNumber	lineNumber
columnNumber	columnNumber
type	"access-to-opener"
Queue body as "coop" for coop's reporting endpoint with coopURL and environment.

To queue a violation report for access from the opener, given a cross-origin opener policy coop, two URLs coopURL and openerURL, two origins coopOrigin and openerOrigin, a string propertyName, and a referrer referrer:

If coop's reporting endpoint is null, return.

Let serializedReferrer be an empty string.

If referrer is a URL, set serializedReferrer to the serialization of referrer.

Let body be a new object containing the following properties:

key	value
disposition	"reporting"
effectivePolicy	coop's report-only value
property	propertyName
openerURL	If coopOrigin and openerOrigin are same origin, this is the sanitization of openerURL, null otherwise.
referrer	serializedReferrer
type	"access-to-opener"
Queue body as "coop" for coop's reporting endpoint with coopURL.

To queue a violation report for access from an opened window, given a cross-origin opener policy coop, three URLs coopURL, openedWindowURL and initialWindowURL, three origins coopOrigin, openedWindowOrigin, and openerInitialOrigin, and a string propertyName:

If coop's reporting endpoint is null, return.

Let body be a new object containing the following properties:

key	value
disposition	"reporting"
effectivePolicy	coopValue
property	coop's report-only value
openedWindowURL	If coopOrigin and openedWindowOrigin are same origin, this is the sanitization of openedWindowURL, null otherwise.
openedWindowInitialURL	If coopOrigin and openerInitialOrigin are same origin, this is the sanitization of initialWindowURL, null otherwise.
type	"access-to-opener"
Queue body as "coop" for coop's reporting endpoint with coopURL.

To queue a violation report for access from another window, given a cross-origin opener policy coop, two URLs coopURL and otherURL, two origins coopOrigin and otherOrigin, and a string propertyName:

If coop's reporting endpoint is null, return.

Let body be a new object containing the following properties:

key	value
disposition	"reporting"
effectivePolicy	coop's report-only value
property	propertyName
otherURL	If coopOrigin and otherOrigin are same origin, this is the sanitization of otherURL, null otherwise.
type	access-to-opener
Queue body as "coop" for coop's reporting endpoint with coopURL.

7.1.4 Cross-origin embedder policies
✔MDN
An embedder policy value is one of three strings that controls the fetching of cross-origin resources without explicit permission from resource owners.

"unsafe-none"
This is the default value. When this value is used, cross-origin resources can be fetched without giving explicit permission through the CORS protocol or the `Cross-Origin-Resource-Policy` header.

"require-corp"
When this value is used, fetching cross-origin resources requires the server's explicit permission through the CORS protocol or the `Cross-Origin-Resource-Policy` header.

"credentialless"
When this value is used, fetching cross-origin no-CORS resources omits credentials. In exchange, an explicit `Cross-Origin-Resource-Policy` header is not required. Other requests sent with credentials require the server's explicit permission through the CORS protocol or the `Cross-Origin-Resource-Policy` header.

Before supporting "credentialless", implementers are strongly encouraged to support both:

Private Network Access
Opaque Response Blocking
Otherwise, it would allow attackers to leverage the client's network position to read non public resources, using the cross-origin isolated capability.

An embedder policy value is compatible with cross-origin isolation if it is "credentialless" or "require-corp".

An embedder policy consists of:

A value, which is an embedder policy value, initially "unsafe-none".

A reporting endpoint string, initially the empty string.

A report only value, which is an embedder policy value, initially "unsafe-none".

A report only reporting endpoint string, initially the empty string.

The "coep" report type is a report type whose value is "coep". It is visible to ReportingObservers.

7.1.4.1 The headers
The `Cross-Origin-Embedder-Policy` and `Cross-Origin-Embedder-Policy-Report-Only` HTTP response headers allow a server to declare an embedder policy for an environment settings object. These headers are structured headers whose values must be token. [STRUCTURED-FIELDS]

The valid token values are the embedder policy values. The token may also have attached parameters; of these, the "report-to" parameter can have a valid URL string identifying an appropriate reporting endpoint. [REPORTING]

The processing model fails open (by defaulting to "unsafe-none") in the presence of a header that cannot be parsed as a token. This includes inadvertent lists created by combining multiple instances of the `Cross-Origin-Embedder-Policy` header present in a given response:

`Cross-Origin-Embedder-Policy`	Final embedder policy value
No header delivered	"unsafe-none"
`require-corp`	"require-corp"
`unknown-value`	"unsafe-none"
`require-corp, unknown-value`	"unsafe-none"
`unknown-value, unknown-value`	"unsafe-none"
`unknown-value, require-corp`	"unsafe-none"
`require-corp, require-corp`	"unsafe-none"
(The same applies to `Cross-Origin-Embedder-Policy-Report-Only`.)

To obtain an embedder policy from a response response and an environment environment:

Let policy be a new embedder policy.

If environment is a non-secure context, then return policy.

Let parsedItem be the result of getting a structured field value with `Cross-Origin-Embedder-Policy` and "item" from response's header list.

If parsedItem is non-null and parsedItem[0] is compatible with cross-origin isolation:

Set policy's value to parsedItem[0].

If parsedItem[1]["report-to"] exists, then set policy's endpoint to parsedItem[1]["report-to"].

Set parsedItem to the result of getting a structured field value with `Cross-Origin-Embedder-Policy-Report-Only` and "item" from response's header list.

If parsedItem is non-null and parsedItem[0] is compatible with cross-origin isolation:

Set policy's report only value to parsedItem[0].

If parsedItem[1]["report-to"] exists, then set policy's endpoint to parsedItem[1]["report-to"].

Return policy.

7.1.4.2 Embedder policy checks
To check a navigation response's adherence to its embedder policy given a response response, a navigable navigable, and an embedder policy responsePolicy:

If navigable is not a child navigable, then return true.

Let parentPolicy be navigable's container document's policy container's embedder policy.

If parentPolicy's report-only value is compatible with cross-origin isolation and responsePolicy's value is not, then queue a cross-origin embedder policy inheritance violation with response, "navigation", parentPolicy's report only reporting endpoint, "reporting", and navigable's container document's relevant settings object.

If parentPolicy's value is not compatible with cross-origin isolation or responsePolicy's value is compatible with cross-origin isolation, then return true.

Queue a cross-origin embedder policy inheritance violation with response, "navigation", parentPolicy's reporting endpoint, "enforce", and navigable's container document's relevant settings object.

Return false.

To check a global object's embedder policy given a WorkerGlobalScope workerGlobalScope, an environment settings object owner, and a response response:

If workerGlobalScope is not a DedicatedWorkerGlobalScope object, then return true.

Let policy be workerGlobalScope's embedder policy.

Let ownerPolicy be owner's policy container's embedder policy.

If ownerPolicy's report-only value is compatible with cross-origin isolation and policy's value is not, then queue a cross-origin embedder policy inheritance violation with response, "worker initialization", owner's policy's report only reporting endpoint, "reporting", and owner.

If ownerPolicy's value is not compatible with cross-origin isolation or policy's value is compatible with cross-origin isolation, then return true.

Queue a cross-origin embedder policy inheritance violation with response, "worker initialization", owner's policy's reporting endpoint, "enforce", and owner.

Return false.

To queue a cross-origin embedder policy inheritance violation given a response response, a string type, a string endpoint, a string disposition, and an environment settings object settings:

Let serialized be the result of serializing a response URL for reporting with response.

Let body be a new object containing the following properties:

key	value
type	type
blockedURL	serialized
disposition	disposition
Queue body as the "coep" report type for endpoint on settings.

7.1.5 Sandboxing
A sandboxing flag set is a set of zero or more of the following flags, which are used to restrict the abilities that potentially untrusted resources have:

The sandboxed navigation browsing context flag
This flag prevents content from navigating browsing contexts other than the sandboxed browsing context itself (or browsing contexts further nested inside it), auxiliary browsing contexts (which are protected by the sandboxed auxiliary navigation browsing context flag defined next), and the top-level browsing context (which is protected by the sandboxed top-level navigation without user activation browsing context flag and sandboxed top-level navigation with user activation browsing context flag defined below).

If the sandboxed auxiliary navigation browsing context flag is not set, then in certain cases the restrictions nonetheless allow popups (new top-level browsing contexts) to be opened. These browsing contexts always have one permitted sandboxed navigator, set when the browsing context is created, which allows the browsing context that created them to actually navigate them. (Otherwise, the sandboxed navigation browsing context flag would prevent them from being navigated even if they were opened.)

The sandboxed auxiliary navigation browsing context flag
This flag prevents content from creating new auxiliary browsing contexts, e.g. using the target attribute or the window.open() method.

The sandboxed top-level navigation without user activation browsing context flag
This flag prevents content from navigating their top-level browsing context and prevents content from closing their top-level browsing context. It is consulted only when the sandboxed browsing context's active window does not have transient activation.

When the sandboxed top-level navigation without user activation browsing context flag is not set, content can navigate its top-level browsing context, but other browsing contexts are still protected by the sandboxed navigation browsing context flag and possibly the sandboxed auxiliary navigation browsing context flag.

The sandboxed top-level navigation with user activation browsing context flag
This flag prevents content from navigating their top-level browsing context and prevents content from closing their top-level browsing context. It is consulted only when the sandboxed browsing context's active window has transient activation.

As with the sandboxed top-level navigation without user activation browsing context flag, this flag only affects the top-level browsing context; if it is not set, other browsing contexts might still be protected by other flags.

The sandboxed origin browsing context flag
This flag forces content into an opaque origin, thus preventing it from accessing other content from the same origin.

This flag also prevents script from reading from or writing to the document.cookie IDL attribute, and blocks access to localStorage.

The sandboxed forms browsing context flag
This flag blocks form submission.

The sandboxed pointer lock browsing context flag
This flag disables the Pointer Lock API. [POINTERLOCK]

The sandboxed scripts browsing context flag
This flag blocks script execution.

The sandboxed automatic features browsing context flag
This flag blocks features that trigger automatically, such as automatically playing a video or automatically focusing a form control.

The sandboxed document.domain browsing context flag
This flag prevents content from using the document.domain setter.

The sandbox propagates to auxiliary browsing contexts flag
This flag prevents content from escaping the sandbox by ensuring that any auxiliary browsing context it creates inherits the content's active sandboxing flag set.

The sandboxed modals flag
This flag prevents content from using any of the following features to produce modal dialogs:

window.alert()
window.confirm()
window.print()
window.prompt()
the beforeunload event
The sandboxed orientation lock browsing context flag
This flag disables the ability to lock the screen orientation. [SCREENORIENTATION]

The sandboxed presentation browsing context flag
This flag disables the Presentation API. [PRESENTATION]

The sandboxed downloads browsing context flag
This flag prevents content from initiating or instantiating downloads, whether through downloading hyperlinks or through navigation that gets handled as a download.

The sandboxed custom protocols navigation browsing context flag
This flag prevents navigations toward non fetch schemes from being handed off to external software.

When the user agent is to parse a sandboxing directive, given a string input, a sandboxing flag set output, it must run the following steps:

Split input on ASCII whitespace, to obtain tokens.

Let output be empty.

Add the following flags to output:

The sandboxed navigation browsing context flag.

The sandboxed auxiliary navigation browsing context flag, unless tokens contains the allow-popups keyword.

The sandboxed top-level navigation without user activation browsing context flag, unless tokens contains the allow-top-navigation keyword.

The sandboxed top-level navigation with user activation browsing context flag, unless tokens contains either the allow-top-navigation-by-user-activation keyword or the allow-top-navigation keyword.

This means that if the allow-top-navigation is present, the allow-top-navigation-by-user-activation keyword will have no effect. For this reason, specifying both is a document conformance error.

The sandboxed origin browsing context flag, unless the tokens contains the allow-same-origin keyword.

The allow-same-origin keyword is intended for two cases.

First, it can be used to allow content from the same site to be sandboxed to disable scripting, while still allowing access to the DOM of the sandboxed content.

Second, it can be used to embed content from a third-party site, sandboxed to prevent that site from opening popups, etc, without preventing the embedded page from communicating back to its originating site, using the database APIs to store data, etc.

The sandboxed forms browsing context flag, unless tokens contains the allow-forms keyword.

The sandboxed pointer lock browsing context flag, unless tokens contains the allow-pointer-lock keyword.

The sandboxed scripts browsing context flag, unless tokens contains the allow-scripts keyword.

The sandboxed automatic features browsing context flag, unless tokens contains the allow-scripts keyword (defined above).

This flag is relaxed by the same keyword as scripts, because when scripts are enabled these features are trivially possible anyway, and it would be unfortunate to force authors to use script to do them when sandboxed rather than allowing them to use the declarative features.

The sandboxed document.domain browsing context flag.

The sandbox propagates to auxiliary browsing contexts flag, unless tokens contains the allow-popups-to-escape-sandbox keyword.

The sandboxed modals flag, unless tokens contains the allow-modals keyword.

The sandboxed orientation lock browsing context flag, unless tokens contains the allow-orientation-lock keyword.

The sandboxed presentation browsing context flag, unless tokens contains the allow-presentation keyword.

The sandboxed downloads browsing context flag, unless tokens contains the allow-downloads keyword.

The sandboxed custom protocols navigation browsing context flag, unless tokens contains either the allow-top-navigation-to-custom-protocols keyword, the allow-popups keyword, or the allow-top-navigation keyword.

Every top-level browsing context has a popup sandboxing flag set, which is a sandboxing flag set. When a browsing context is created, its popup sandboxing flag set must be empty. It is populated by the rules for choosing a navigable and the obtain a browsing context to use for a navigation response algorithm.

Every iframe element has an iframe sandboxing flag set, which is a sandboxing flag set. Which flags in an iframe sandboxing flag set are set at any particular time is determined by the iframe element's sandbox attribute.

Every Document has an active sandboxing flag set, which is a sandboxing flag set. When the Document is created, its active sandboxing flag set must be empty. It is populated by the navigation algorithm.

Every CSP list cspList has CSP-derived sandboxing flags, which is a sandboxing flag set. It is the return value of the following algorithm:

Let directives be an empty ordered set.

For each policy in cspList:

If policy's disposition is not "enforce", then continue.

If policy's directive set contains a directive whose name is "sandbox", then append that directive to directives.

If directives is empty, then return an empty sandboxing flag set.

Let directive be directives[directives's size − 1].

Return the result of parsing the sandboxing directive directive.

To determine the creation sandboxing flags for a browsing context browsing context, given null or an element embedder, return the union of the flags that are present in the following sandboxing flag sets:

If embedder is null, then: the flags set on browsing context's popup sandboxing flag set.

If embedder is an element, then: the flags set on embedder's iframe sandboxing flag set.

If embedder is an element, then: the flags set on embedder's node document's active sandboxing flag set.

7.1.6 Policy containers
A policy container is a struct containing policies that apply to a Document, a WorkerGlobalScope, or a WorkletGlobalScope. It has the following items:

A CSP list, which is a CSP list. It is initially empty.

An embedder policy, which is an embedder policy. It is initially a new embedder policy.

A referrer policy, which is a referrer policy. It is initially the default referrer policy.

Move other policies into the policy container.

To clone a policy container given a policy container policyContainer:

Let clone be a new policy container.

For each policy in policyContainer's CSP list, append a copy of policy into clone's CSP list.

Set clone's embedder policy to a copy of policyContainer's embedder policy.

Set clone's referrer policy to policyContainer's referrer policy.

Return clone.

To determine whether a URL url requires storing the policy container in history:

If url's scheme is "blob", then return false.

If url is local, then return true.

Return false.

To create a policy container from a fetch response given a response response and an environment-or-null environment:

If response's URL's scheme is "blob", then return a clone of response's URL's blob URL entry's environment's policy container.

Let result be a new policy container.

Set result's CSP list to the result of parsing a response's Content Security Policies given response.

If environment is non-null, then set result's embedder policy to the result of obtaining an embedder policy given response and environment. Otherwise, set it to "unsafe-none".

Set result's referrer policy to the result of parsing the `Referrer-Policy` header given response. [REFERRERPOLICY]

Return result.

To determine navigation params policy container given a URL responseURL and four policy container-or-nulls historyPolicyContainer, initiatorPolicyContainer, parentPolicyContainer, and responsePolicyContainer:

If historyPolicyContainer is not null, then:

Assert: responseURL requires storing the policy container in history.

Return a clone of historyPolicyContainer.

If responseURL is about:srcdoc, then:

Assert: parentPolicyContainer is not null.

Return a clone of parentPolicyContainer.

If responseURL is local and initiatorPolicyContainer is not null, then return a clone of initiatorPolicyContainer.

If responsePolicyContainer is not null, then return responsePolicyContainer.

Return a new policy container.

To initialize a worker global scope's policy container given a WorkerGlobalScope workerGlobalScope, a response response, and an environment environment:

If workerGlobalScope's url is local but its scheme is not "blob":

Assert: workerGlobalScope's owner set's size is 1.

Set workerGlobalScope's policy container to a clone of workerGlobalScope's owner set[0]'s relevant settings object's policy container.

Otherwise, set workerGlobalScope's policy container to the result of creating a policy container from a fetch response given response and environment.

← 6.12 The popover attribute — Table of Contents — 7.2 APIs related to navigation and session history →