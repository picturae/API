# OpenSKOS 2 RESTful API

## TODO

- [X] SPARQL paging experiments
  - [X] use LIMIT and OFFSET
  - [ ] use a last seen concept (DECISION: No)
  - [/] can we use the [hydra paging (`hydra:PartialCollectionView`)](http://www.hydra-cg.com/spec/latest/core/#collections)? (envisioned part of the NDE generic API)
- [ ] how to handle URIs
  - e.g., register prefix in `application.ini` (see [prefixes](#prefixes)
  - user specified relation types
  - user provided properties
  - refering to OpenSKOS resources by their foreign URIs
- [X] check if we follow RFC 6570 correctly (NO)
- [X] what do we use for refering to internal resources
  - [X] `uri` or `id` or both
  - [ ] allow prefixes (see above)
- [ ] add cardinalities
- [X] if an higher level includes concepts, e.g., for collections and concept schemes, also include the concept `<projection params>.props`? NO
- [X] camelCase in params, e.g., `conceptScheme`, but not in endpoints, e.g., `.../conceptscheme`? OK as it is
- [ ] check how set and institution are expressed in OpenSKOS 2, e.g., `openskos:set` or `openskos:institution` (see concept `<projection params>`)?
- [X] analyze if major (read) functionality of the editor is covered in an appropriate amount of requests, e.g., a list should not require a call for each concept
- [/] return codes
  - 200 everything OK
  - 400 if other parameters have a clash with search profile
  - 401 or 403 if unauthorized
  - 404 a resource is not found
  - 405 method not allowed, e.g., a PUT on a list
  - 406 unsupported RDF serialization
  - 500 and higher: oops
- [X] return representations (at least JSON-LD, RDF/XML, TriG/TriX, (incl. Hydra paging vocab), HTML)
- [X] support for dates in projection, selection/filter and order
  - submitted
  - modified
  - accepted
  - openskos:deleted
  - NOTE: SPARQL has no support for xsd:duration, but does have support for operators on xsd:dateTime, so some (all?) ranges can be translated into boolean queries using operators (`2018` => `x >= xsd:dateTime('1-1-2018') and x <= xsd:dateTime('31-12-2018')`)
  - NOTE: maybe support some shortcuts, e.g., today, yesterday, this week, last week, this month, last month, this year, last year
- [X] support for status in projection, selection/filter and order (?)
- [X] support for user in projection, selection/filter and order (?)
  - creator
  - openskos:modifiedBy
  - openskos:acceptedBy
  - openskos:deletedBy
- [X] support for search profiles
- [/] concepts is used as the main discussion endpoint, sync with other endpoints
- [X] support for SKOS-XL labels
- [ ] check consistent use of institution vs tenant
- [ ] discuss the first class citizenship of `skosxl:Labels`
- [X] add format parameter
- [ ] lists of many (all?) resources are filtered based on access rights, e.g., admin can see all, users can only see what belongs to their institution (or their searchProfile allows)

## Foundation

* [X] content negotiation
  * [X] also a parameter (?format=)
  * [ ] incl. HTML (containing the same info as the RDF serializations)
  * [X] JSON-LD instead of proprietary JSON
  * [/] support at least JSON-LD, RDF/XML, TriG/TriX, (incl. Hydra paging vocab), HTML
  * [X] all should return the same info, which is controlled (potentially) by the projection parameters
* [ ] API key in header and use HTTPS always
  * good for security, bad for traceability (look at server logs and see who has done what)
* [X] focus on read-only first
* [ ] hide implementation details (Lucene query language)
* [ ] using just OpenSKOS generated URIs should work out of the box
* [ ] using foreign URIs (external SKOS imported into OpenSKOS) should work out of the box (just don't use id endpoints)

## Does the new API remedy critisism on the old API?
- [X] the API should identify **resources** instead of **actions** (TODO: maybe autocomplete is still too action like)
- [/] use the same URI for all HTTP CRUD methods (NOTE: currently the API only defines the GET methods, but the same (singular) endpoints should support the CRUD methods in the future. POSTs could happen on the plural endpoints as well.)
- [/] support content negotiation, so you have neat (independent of format) conceptual resource URIs, i.e., not representation specific URIs (NOTE: a foundation of the new API is support content negotiation. However, support for specifying format in a uri part breaks this, so only support content negotiation or a format parameter!)
- [/] client can interact choosing one of the supported RDF representations (NOTE: for the CRUD operations all RDF serializations supported for GET should also be supported for PUT and POST. All representations should contain the same information! HTML is only supported for GET.)
- [X] describe the openskos namespace, so the responses are self describing
- [/] don't pass API keys in the body (NOTE: API keys should always be passed in HTTPS based requests)
- [X] use JSON-LD, propriety JSON is not self descriptive 
- [/] HATEOAS (NOTE: there might not be URIs just literals for some entities (e.g., NAA) due to legacy)
- [/] HTML representations should provide HATEOAS links
- [?] replace API keys by OAuth(2)
- [?] use the Linked Data Platform (NOTE: at least paging collides with Hydra paging)
- [?] use Triple Pattern Fragments

## Does the new API provide good coverage for the functionality provided by the editor?

- [X] check if the backend can be worked with
  1. `.../ping`
- [X] get the user profile
  1. `.../user`
- [X] list concept schemes
  1. `.../conceptschemes?level=1&institions=<id>`
  1. or with a search profile: `.../conceptschemes?level=1&searchProfile=<id>`
- [X] list concepts in a concept scheme
  1. `.../concepts?conceptScheme=<id or URI>&props=prefLabel`
- [X] show a selected concept and its relations
  1. info about the concept: `.../concept/<id>?level=2`
  2. relations from the concept: `.../relations?subject=<id>&types=semanticRelation&props=prefLabel`
  3. relations to the concept: `.../relations?object=<id>&types=semanticRelation&props=prefLabel`
  4. user specific relations to the concept: `.../relations?object=<id>&types=mi:fasterThan&props=prefLabel`
- [X] search for a concept
  1. `.../concepts?text=<search terms>&props=prefLabel`
  1. or with a search profile: `.../concepts?text=<search terms>&props=prefLabel&searchProfile=<id>`
- [X] list of search profiles
  1. `.../searchprofiles`
- [x] info of a search profile
  1. `.../searchprofile/<id>`
  2. list of concept schemes: `.../conceptschemes?level=1`
  3. list of sets: `.../sets?level=1`
  4. list of institutions: `.../institutions?level=1`
- [X] get the info on the institution
  1. `.../institution/<id>?level=1` (NOTE: id is known from the user profile) (TODO: check if a higher level is needed due to VCard)
- [X] get the list of sets
  1. `.../sets?institutions=<id>&level=1`
- [X] get the info on a set
  1. `.../set/<id>?level=1` (TODO: check if a higher level is needed due to VCard)
- [X] get the list of concept schemes
  1. `.../conceptschemes?institutions=<id>&level=1`
- [X] get the info on a concept scheme
  1. `.../conceptscheme/<id>?level=1`

## Notation

The API endpoints are described using [URI templates](http://www.rfc-editor.org/info/rfc6570). However, sets of parameters are pretty common, we name these groups and use `<parameter group name>` to refer to them in an URI.

## common variables

* {uri}=`the foreign URI of an resource` 
* {id}=`the native ID of an resource`
* {lang}=`known language code (BCP 47)`

## institutions

Institutions have the rdf:type <http://www.w3.org/ns/org#FormalOrganization>

Institutions contain the following internally used predicates:

<http://www.w3.org/2006/vcard/ns#ADR>
<http://www.w3.org/2006/vcard/ns#Country>
<http://www.w3.org/2006/vcard/ns#EMAIL>
<http://www.w3.org/2006/vcard/ns#Locality>
<http://www.w3.org/2006/vcard/ns#Pcode>
<http://www.w3.org/2006/vcard/ns#orgunit>
<http://openskos.org/xmlns#code>
<http://openskos.org/xmlns#disableSearchInOtherTenants>
<http://openskos.org/xmlns#enableStatussesSystem>
<http://openskos.org/xmlns#enableskosxl>
<http://openskos.org/xmlns#name>
<http://openskos.org/xmlns#uuid>
<http://openskos.org/xmlns#webpage>


`.../institutions?<query params>&<limit params>`

`.../institution?uri={uri}&<query params>` (foreign uri)

`.../institution/{id}?<query params>` (native uri)

`{id}` will be applied to, in order of precedence: <http://openskos.org/xmlns#uuid>, <http://openskos.org/xmlns#code> 

`<query params>`
* level:
  * `1`: data properties of the institution (default)
  * `2`: + object properties of the institution [On Hold]
  * `3`: + data properties of the set [On Hold]
  * `4`: + object properties of the set [On Hold]

NOTE: the OpenSKOS editor should never use level `3` or `4`, they're more interesting for external LD parties


`<filter params>`

*Institutions are the top level objects in the SKOS hierachy, and have no filters.*

`<limit params>`
* limit=`nr of institutions to return`
* offset=`offset seen institution`



Return codes

  - 200 Institution found and data returned
  - 400 Attempted to filter on un-usable predicate
  - 400 Unusable limit or level parameters
  - 404 No resource could be matched
  - 406 unsupported RDF serialization
  - 500 Unexpected error

## sets

Sets have the rdf:type <http://openskos.org/xmlns#set>

Sets contain the following internally used predicates:

<http://purl.org/dc/terms/description>
<http://purl.org/dc/terms/license>
<http://purl.org/dc/terms/publisher>
<http://purl.org/dc/terms/title>
<http://openskos.org/xmlns#allow_oai>
<http://openskos.org/xmlns#code>
<http://openskos.org/xmlns#conceptBaseUri>
<http://openskos.org/xmlns#oai_baseURL>
<http://openskos.org/xmlns#tenant>
<http://openskos.org/xmlns#webpage>


`.../sets?<query params>&<filter params>&<limit params>`

`.../set/{id}?<query params>` (native uri)

`{id}` will be applied to, in order of precedence: <http://openskos.org/xmlns#code> 

`.../set?uri={uri}&<query params>` (foreign uri)

@TODO: Within OpenSkos, sets seemingly do not have uuids. Should this be applied?


`<query params>`
* level:
  * `1`: data properties of the set (default)
  * `2`: + object properties of the set [On Hold]
  * `3`: + data properties of the concept scheme or collection [On Hold]
  * `4`: + object properties of the concept scheme or collection [On Hold]

NOTE: the OpenSKOS editor should never use level `3` or `4`, they're more interesting for external LD parties

`<filter params>`
* institutions=`comma separated list of institution URIs or IDs`. Applied to <http://openskos.org/xmlns#tenant>

@TODO: Why is the set predicate <http://openskos.org/xmlns#tenant> equivalent to the tenant predicate <http://openskos.org/xmlns#code> ?

`<limit params>`
* limit=`nr of sets to return`
* offset=`offset of last seen set` (NOTE: starts at 0)


Return codes

  - 200 Set found and data returned
  - 400 Attempted to filter on un-usable predicate
  - 400 Unusable limit or level parameters
  - 404 No resource could be matched
  - 406 unsupported RDF serialization
  - 500 Unexpected error

## concept schemes

ConceptSchemes have the rdf:type <http://www.w3.org/2004/02/skos/core#ConceptScheme>

Concept-Schemes may contain the following internally used predicates:

<http://www.w3.org/2004/02/skos/core#hasTopConcept>
<http://purl.org/dc/elements/1.1/creator>
<http://purl.org/dc/terms/creator>
<http://purl.org/dc/terms/dateSubmitted>
<http://purl.org/dc/terms/description>
<http://purl.org/dc/terms/modified>
<http://purl.org/dc/terms/title>
<http://openskos.org/xmlns#dateDeleted>
<http://openskos.org/xmlns#deletedBy>
<http://openskos.org/xmlns#modifiedBy>
<http://openskos.org/xmlns#set>
<http://openskos.org/xmlns#status>
<http://openskos.org/xmlns#tenant>
<http://openskos.org/xmlns#uuid>

`.../conceptschemes?<query params>&<filter params>&<limit params>`

`.../conceptscheme/{id}?<query params>` (native uri)

`{id}` will be applied to, in order of precedence: <http://openskos.org/xmlns#uuid>

`.../conceptscheme?uri={uri}&<query params>` (foreign uri)

`<query params>`
* level:
  * `1`: data properties of the concept scheme (default)
  * `2`: + object properties of the concept scheme [On Hold]
  * `3`: + data properties of the concept [On Hold]
  * `4`: + object properties of the concept [On Hold]

NOTE: On large datasets, levels 3 and 4 could impose a significant performance hit

NOTE: the OpenSKOS editor should never use level `3` or `4`, they're more interesting for external LD parties

`<filter params>`
* institutions=`comma separated list of institution URIs or IDs`. Applied to <http://openskos.org/xmlns#tenant>.
* sets=`comma separated list of set URIs or IDs`. Applied to <http://openskos.org/xmlns#set> 
* searchProfile=`id of a search profile`. Stored in MySQL table 'search_profiles'.

`<limit params>`
* limit=`nr of concept schemes to return`
* offset=`offset of last seen concept scheme` (NOTE: starts at 0)


Return codes

  - 200 Concept Scheme found and data returned
  - 400 Attempted to filter on un-usable predicate
  - 400 Unusable limit or level parameters
  - 404 No resource could be matched
  - 406 unsupported RDF serialization
  - 500 Unexpected error

## collections

@TODO: This section is not complete. Awaiting further details from Meertens.

Collections have the rdf:type's <http://www.w3.org/2004/02/skos/core#Collection> and <http://www.w3.org/2004/02/skos/core#OrderedCollection> (awaiting further details from Meertens)

It is very possible that Meertens' data is 100% compliant with: https://www.w3.org/TR/skos-reference/#collections , but I cannot confirm that at present


`.../collections?<query params>&<filter params>&<limit params>`

`.../collection/{id}?<query params>` (native uri)

`.../collection?uri={uri}&<query params>` (foreign uri)

`<query params>`
* level:
  * `1`: data properties of the collection (default)
  * `2`: + object properties of the collection
  * `3`: + data properties of the concept
  * `4`: + object properties of the concept

NOTE: On large datasets, levels 3 and 4 could impose a significant performance hit

NOTE: the OpenSKOS editor should never use level `3` or `4`, they're more interesting for external LD parties

`<filter params>`
* institutions=`comma separated list of institution URIs or IDs`
* sets=`comma separated list of set URIs or IDs`
* searchProfile=`id of a search profile`. Stored in MySQL table 'search_profiles'.

`<limit params>`
* limit=`nr of collections to return`
* offset=`offset of last seen collection` (NOTE: starts at 0)

Return codes

  - 200 Collection found and data returned
  - 400 Attempted to filter on un-usable predicate
  - 400 Unusable limit or level parameters
  - 404 No resource could be matched
  - 406 unsupported RDF serialization
  - 500 Unexpected error

## concepts

Concepts have the rdf:type <http://www.w3.org/2004/02/skos/core#Concept>

	
Concept may contain the following internally used predicates:

<http://www.w3.org/2004/02/skos/core#inScheme>
<http://purl.org/dc/elements/1.1/contributor>
<http://purl.org/dc/elements/1.1/creator>
<http://purl.org/dc/terms/contributor>
<http://purl.org/dc/terms/creator>
<http://purl.org/dc/terms/dateAccepted>
<http://purl.org/dc/terms/dateApproved>
<http://purl.org/dc/terms/dateSubmitted>
<http://purl.org/dc/terms/modified>
<http://purl.org/dc/terms/publisher>
<http://purl.org/dc/terms/title>
<http://openskos.org/xmlns#acceptedBy>
<http://openskos.org/xmlns#dateDeleted>
<http://openskos.org/xmlns#deletedBy>
<http://openskos.org/xmlns#modifiedBy>
<http://openskos.org/xmlns#notation>
<http://openskos.org/xmlns#set>
<http://openskos.org/xmlns#status>
<http://openskos.org/xmlns#tenant>
<http://openskos.org/xmlns#toBeChecked>
<http://openskos.org/xmlns#uuid>
<http://www.w3.org/2004/02/skos/core#notation>
<http://www.w3.org/2004/02/skos/core#Note>
<http://www.w3.org/2004/02/skos/core#note>
<http://www.w3.org/2004/02/skos/core#changeNote>
<http://www.w3.org/2004/02/skos/core#scopeNote>
<http://www.w3.org/2004/02/skos/core#historyNote>
<http://www.w3.org/2004/02/skos/core#prefLabel>
<http://www.w3.org/2004/02/skos/core#altLabel>
<http://www.w3.org/2004/02/skos/core#hiddenLabel>
<http://www.w3.org/2004/02/skos/core#example>
<http://www.w3.org/2008/05/skos-xl#altLabel>
<http://www.w3.org/2008/05/skos-xl#hiddenLabel>
<http://www.w3.org/2008/05/skos-xl#prefLabel>


Concepts will also contain the following semantic matches. Some of these are transitive matches, applied via Jena reasoners.

<http://www.w3.org/2004/02/skos/core#broadMatch>
<http://www.w3.org/2004/02/skos/core#broader>
<http://www.w3.org/2004/02/skos/core#broaderMatch>
<http://www.w3.org/2004/02/skos/core#broaderTransitive>
<http://www.w3.org/2004/02/skos/core#definition>
<http://www.w3.org/2004/02/skos/core#editorialNote>
<http://www.w3.org/2004/02/skos/core#exactMatch>
<http://www.w3.org/2004/02/skos/core#closeMatch>
<http://www.w3.org/2004/02/skos/core#narrowMatch>
<http://www.w3.org/2004/02/skos/core#narrower>
<http://www.w3.org/2004/02/skos/core#narrowerTransitive>
<http://www.w3.org/2004/02/skos/core#related>
<http://www.w3.org/2004/02/skos/core#relatedMatch>
<http://www.w3.org/2004/02/skos/core#topConceptOf>

NOTE: For performance reasons: concepts are searched first in Solr; once matches are found, further filtering is applied via Jena records.

NOTE: Concepts are connected to their relevant Concept Schemes by the Predicate <http://www.w3.org/2004/02/skos/core#inScheme>, and not the rdf:type of the Concept scheme. The connection is a many-to-many relotionship.

`.../concepts?<selection params>[?]&<filter params>&<projection params>&<order params>&<limit params>&<format params>`

`.../concept/{id}?<projection params>` (native uri)

`.../concept?uri={uri}&<projection params>` (foreign uri)

`<selection params>`
* text[1]=`search terms`
  * support for
    * strings: ab c
    * wildcard: \*a a\* a*b
    * boolean: a AND b OR NOT c
    * brackets: (a OR b) AND c
    * phrases: "ab c"
    * escapes: \\" \\* \\\\
* fields[?]=`comma separated list of`
  * default: label
  * label(@{lang})?
    * prefLabel(@{lang})?
    * altLabel(@{lang})?
    * hiddenLabel(@{lang})?
  * note(@{lang})?
    * definition(@{lang})?
    * example(@{lang})?
    * changeNote(@{lang})?
    * historyNote(@{lang})?
    * scopeNote(@{lang})?
    * editorialNote(@{lang})?
  * notation


`<filter params>`
* institutions=`comma separated list of institution URIs or IDs`. Applied to <http://openskos.org/xmlns#tenant>
* sets=`comma separated list of set URIs or IDs`. Applied to <http://openskos.org/xmlns#set>
* conceptSchemes=`comma separated list of concept scheme URIs or IDs`. Applied to <http://www.w3.org/2004/02/skos/core#inScheme>
* collections=`comma separated list of collection URIs or IDs` [On Hold: Predicate not known]
* searchProfile=`id of a search profile`. Stored in MySQL table 'search_profiles'.
* dateSubmitted=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/dateSubmitted> 
* modified=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/modified>
* dateAccepted=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/dateAccepted>
* openskos:deleted=`xsd:duration and some shortcuts (?)`. Applied to  <http://openskos.org/xmlns#dateDeleted>
* statuses=`comma separated list of statuses`. Applied to <http://openskos.org/xmlns#status>
  * `candidate`
  * `approved`
  * `redirected`
  * `not_compliant`
  * `rejected`
  * `obsolete`
  * `deleted`
* creator=`comma separated list of user URIs or IDs`. Applied to <http://purl.org/dc/terms/creator>
* openskos:modifiedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#modifiedBy>
* openskos:acceptedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#acceptedBy>
* openskos:deletedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#deletedBy>

NOTE: There are no semantic relations in the filters. The correct way to access these is to recall the concept using level 2, or to use the `.../relations` endpoint

`<projection params>`
* level=`which properties to return`
  * `1`: data properties of the concepts (default)
  * `2`: + object properties of the concepts [These must be implemented. NOT on-hold]
* props=`comma separated list of`
  * default: uri,prefLabel,definition
  * all
  * uri
  * label(@{lang})?
    * prefLabel(@{lang})?
    * altLabel(@{lang})?
    * hiddenLabel(@{lang})?
  * note(@{lang})?
    * definition(@{lang})?
    * example(@{lang})?
    * changeNote(@{lang})?
    * historyNote(@{lang})?
    * scopeNote(@{lang})?
    * editorialNote(@{lang})?
  * notation
  * semanticRelation
    * broader
    * narrower
    * related
    * broaderTransitive
    * narrowerTransitive
    * mappingRelation
      * closeMatch
      * exactMatch
      * broadMatch
      * narrowMatch
      * relatedMatch
  * inScheme
    * topConceptOf
  * openskos:inCollection (OpenSKOS specific inverse of skos:member) [NOTE: currently inSkosCollection]
  * openskos:status
  * openskos:set [TODO: check]
  * openskos:institution [TODO: check]
  * dates
    * dateSubmitted
    * modified
    * dateAccepted
    * openskos:deleted
  * users
    * creator
    * openskos:modifiedBy
    * openskos:acceptedBy
    * openskos:deletedBy
  * {user relation types}
  * {user properties}
* lang=`comma separated list of known language codes (BCP 47)`
  * if not specified all languages are projected (default)
* skosxl=`yes or no` (default: follow configuration of the tenant)
  NOTE: `skosxl=yes` can result in a `400` if one of the tenants doesn't support SKOS-XL

`<order params>`
* order=`comma separated list of`
  * prefLabel(@{lang})?(:(asc|desc))? (default direction: asc)
  * altLabel(@{lang})?(:(asc|desc))? (default direction: asc)
  * hiddenLabel(@{lang})?(:(asc|desc))? (default direction: asc)
  * notation(:(asc|desc))? (default direction: asc)
  * uri(:(asc|desc))? (??) (default direction: asc)
  * score(:(asc|desc))? (default) (default direction: desc)
  * institution(:(asc|desc))? (default direction: asc)
  * set(:(asc|desc))? (default direction: asc)
  * conceptScheme(:(asc|desc))? (default direction: asc)
  * collection(:(asc|desc))? (default direction: asc)
  * dateSubmitted(:(asc|desc))? (default direction: asc)
  * modified(:(asc|desc))? (default direction: asc)
  * dateAccepted(:(asc|desc))? (default direction: asc)
  * openskos:deleted(:(asc|desc))? (default direction: asc)

`<limit params>`
* limit=`nr of concepts to return`
* offset=`offset of last seen concept` (NOTE: starts at 0)


Return codes

  - 200 Concept Scheme found and data returned
  - 400 Attempted to filter on un-usable predicate
  - 400 Attempted to use non-valid projection/selection or order fields
  - 400 Unreadable values in  projection/selection or order values
  - 400 Unusable limit or level parameters
  - 400 Attempted to recall XL labels on a tenant that does not support SkosXL
  - 404 No resource could be matched
  - 406 unsupported RDF serialization
  - 500 Unexpected error

## autocomplete

`.../concepts/autocomplete?<selection params>&<projection params>&<filter params>`


NOTE: For performance reasons: concepts are searched first in Solr; once matches are found, further filtering is applied via Jena records.

`<selection params>`
* text=`substring to match case insensitively`
* fields=`comma separated list of`
  * default: label
  * label(@{lang})?
    * prefLabel(@{lang})?
    * altLabel(@{lang})?
    * hiddenLabel(@{lang})?
  * notation

`<projection params>`
* props=`comma separated list of`
  * default: uri,prefLabel
  * uri
  * label(@{lang})?
    * prefLabel(@{lang})?
    * altLabel(@{lang})?
    * hiddenLabel(@{lang})?
  * notation

`<filter params>`
* institutions=`comma separated list of institution URIs or IDs`. Applied to <http://openskos.org/xmlns#tenant>
* sets=`comma separated list of set URIs or IDs`. Applied to <http://openskos.org/xmlns#set>
* conceptSchemes=`comma separated list of concept scheme URIs or IDs`. Applied to <http://www.w3.org/2004/02/skos/core#inScheme>
* collections=`comma separated list of collection URIs or IDs` [On Hold: Predicate not known]
* searchProfile=`id of a search profile`. Stored in MySQL table 'search_profiles'.
* dateSubmitted=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/dateSubmitted> 
* modified=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/modified>
* dateAccepted=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/dateAccepted>
* openskos:deleted=`xsd:duration and some shortcuts (?)`. Applied to  <http://openskos.org/xmlns#dateDeleted>
* statuses=`comma separated list of statuses`. Applied to <http://openskos.org/xmlns#status>
  * `candidate`
  * `approved`
  * `redirected`
  * `not_compliant`
  * `rejected`
  * `obsolete`
  * `deleted`
* creator=`comma separated list of user URIs or IDs`. Applied to <http://purl.org/dc/terms/creator>
* openskos:modifiedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#modifiedBy>
* openskos:acceptedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#acceptedBy>
* openskos:deletedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#deletedBy>


NOTE: Menzo requested the following, but it is output configuration and not a filter parameter
> * skosxl=`yes or no` (default: follow configuration of the tenant)
> * TODO: is this nice?

`.../labels/autocomplete?<selection params>&<projection params>&<filter params>`

`<selection params>`
* text=`substring to match case insensitively`

`<projection params>`
* props=`comma separated list of`
  * default: uri,literalForm
  * uri
  * literalForm

`<filter params>`
* institutions=`comma separated list of institution URIs or IDs`. Applied to <http://openskos.org/xmlns#tenant>
* sets=`comma separated list of set URIs or IDs`. Applied to <http://openskos.org/xmlns#set>
* conceptSchemes=`comma separated list of concept scheme URIs or IDs`. Applied to <http://www.w3.org/2004/02/skos/core#inScheme>
* collections=`comma separated list of collection URIs or IDs` [On Hold: Predicate not known]
* searchProfile=`id of a search profile`. Stored in MySQL table 'search_profiles'.
* dateSubmitted=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/dateSubmitted> 
* modified=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/modified>
* dateAccepted=`xsd:duration and some shortcuts (?)`. Applied to <http://purl.org/dc/terms/dateAccepted>
* openskos:deleted=`xsd:duration and some shortcuts (?)`. Applied to  <http://openskos.org/xmlns#dateDeleted>
* statuses=`comma separated list of statuses`. Applied to <http://openskos.org/xmlns#status>
  * `candidate`
  * `approved`
  * `redirected`
  * `not_compliant`
  * `rejected`
  * `obsolete`
  * `deleted`
* creator=`comma separated list of user URIs or IDs`. Applied to <http://purl.org/dc/terms/creator>
* openskos:modifiedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#modifiedBy>
* openskos:acceptedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#acceptedBy>
* openskos:deletedBy=`comma separated list of user URIs or IDs`. Applied to <http://openskos.org/xmlns#deletedBy>

@TODO: lots of these bookkeeping-based filters depend on SKOS-XL labels being first class citizens!

@TODO: There is currently no specification for the HTTP return codes for the _autocomplete_. To be discussed. 

## labels

> NOTE over SKOS-XL
> 
> Skos-XL labels are formally not first-class citizens in OpenSkos. Implementing the structure desribed here would involve changes to the existing openskos editor and structure.
> This is currenly not budgeted for
> 
> B.Hillier

`.../labels?<selection params>&<filter params>&<projection params>&<order params>&<limit params>`

NOTE: maybe only when `skosxl:label` are first class citizens

`.../label/{id}?<projection params>` (native uri)

`.../label?uri={uri}&<projection params>` (foreign uri)

`<projection params>`
* level=`which properties to return`
  * `1`: data properties of the concepts (default)
  * `2`: + object properties of the concepts
* props=`comma separated list of`
  * default: uri,literalForm
  * all
  * uri
  * literalForm(@{lang})?
  * openskos:isPrefLabelOf (inverse of prefLabel)
  * openskos:isAltLabelOf (inverse of altLabel)
  * openskos:isHiddenLabelOf (inverse of hiddenLabel)
  * labelRelation
  * openskos:set [TODO: check]
  * openskos:institution (NOTE: is currently openskos:tenant)
  * status
    * NOTE: not supported at the moment, TODO discuss if labels should be first class citizens
    * "`alive`"
    * `deleted`
  * dates
    * modified
    * NOTE: maybe also have?
      * dateSubmitted
      * openskos:deleted
  * users
    * NOTE: not supported at the moment, TODO discuss if labels should be first class citizens
    * creator
    * openskos:modifiedBy
    * openskos:acceptedBy
    * openskos:deletedBy
  * {user relation types}
  * {user properties}
* lang=`comma separated list of known language codes (BCP 47)`
  * if not specified all languages are projected (default)

## relations

`.../relations?<selection params>&<projection params>&<limit params>`

`<selection params>`
* subject=`uri`
* types=`comma separated list of relation types`
  * semanticRelation
    * broader . Applied to <http://www.w3.org/2004/02/skos/core#broader>
    * narrower. Applied to <http://www.w3.org/2004/02/skos/core#narrower>
    * related. Applied to <http://www.w3.org/2004/02/skos/core#related>
    * broaderTransitive. Applied to <http://www.w3.org/2004/02/skos/core#broaderTransitive>
    * narrowerTransitive. Applied to <http://www.w3.org/2004/02/skos/core#narrowerTransitive>
    * mappingRelation
      * closeMatch. Applied to <http://www.w3.org/2004/02/skos/core#closeMatch>
      * exactMatch. Applied to <http://www.w3.org/2004/02/skos/core#exactMatch>
      * broadMatch. Applied to <http://www.w3.org/2004/02/skos/core#broadMatch>
      * narrowMatch. Applied to <http://www.w3.org/2004/02/skos/core#narrowMatch>
      * relatedMatch. Applied to <http://www.w3.org/2004/02/skos/core#relatedMatch>
  * inScheme
    * topConceptOf. Applied to <http://www.w3.org/2004/02/skos/core#topConceptOf>
  * isReplacedBy (NOTE: currently not yet a triple in OpenSKOS)
  * replaces (NOTE: currently not yet a triple in OpenSKOS)
  * openskos:inCollection (OpenSKOS specific inverse of skos:member) [NOTE: currently inSkosCollection]
  * openskos:inSet (NOTE: currently openskos:set)
  * openskos:tenant. Applied to <http://openskos.org/xmlns#tenant>
  * member. [On hold. Waiting for details of Skos Collections]
  * hasTopConcept. Applied to <http://www.w3.org/2004/02/skos/core#hasTopConcept>
  * prefLabel. Applied to <http://www.w3.org/2004/02/skos/core#prefLabel>
  * altLabel. Applied to <http://www.w3.org/2004/02/skos/core#altLabel>
  * hiddenLabel. Applied to <http://www.w3.org/2004/02/skos/core#hiddenLabel>
  * openskos:isPrefLabelOf (inverse of prefLabel) [On Hold. Currently not part of OpenSkos]
  * openskos:isAltLabelOf (inverse of altLabel) [On Hold. Currently not part of OpenSkos]
  * openskos:isHiddenLabelOf (inverse of hiddenLabel) [On Hold. Currently not part of OpenSkos]
  * labelRelation [@TODO. What is meant here]
  * {user relation types} [TODO: how to handle URIs, e.g., register prefix in application.ini]
* object=`uri`

`<projection params>`
* props=`comma separated list of`
  * default: uri
  * uri
  * title(@{lang})?
  * label(@{lang})?
    * prefLabel(@{lang})?
    * altLabel(@{lang})?
    * hiddenLabel(@{lang})?

`<filter params>`
TODO: what does it mean for subject and object?

`<limit params>`
* limit=`nr of subjects to return`
* offset=`offset of last seen subject` (NOTE: starts at 0)

Return codes

  - 200 Set found and data returned
  - 400 Attempted to filter on un-usable predicate (all of them)
  - 400 Unusable limit or level parameters
  - 400 Attempted to use non-valid projection/selection or order fields
  - 400 Unreadable values in  projection/selection or order values
  - 404 No resource could be matched
  - 406 unsupported RDF serialization
  - 500 Unexpected error

## relation types

`.../relationtypes`

`.../relationtype/{id}` (native uri)

`.../relationtype?uri={uri}` (foreign uri)

## users

`.../users?<query params>&<limit params>`

`.../user/{id}`

`.../user`

NOTE: the user endpoint without ID can be used to retrieve the profile of the logged in user, i.e., identified by her credentials or API key.

NOTE: these endpoints will only return info if the user has admin rights, or asks for her own profile.

`<query params>`
* level:
  * `1`: data properties of the user (default)
  * `2`: + object properties of the user

`<limit params>`
* limit=`nr of institutions to return`
* offset=`offset of last seen user` (NOTE: starts at 0)

## roles

`.../roles`

## search profiles

`.../searchprofiles`

`.../searchprofile/{id}`

NOTE: conditions in a search profile might potentially clash with other selection/filter parameters.

## statuses

`.../statuses`

NOTE: GET only, as the list of statuses is fixed

## prefixes

`.../prefixes`

NOTE: the namespaces are currently hardcoded (not in the MySQL anymore or in `application.ini`), so to make the set extensible (again) will require some work. Extension might only be possible via `application.ini`, so the endpoint could be only a GET.

## ping

`.../ping`

Gives info about the OpenSKOS instance, e.g., version, health.

## vocab

`.../vocab`

Gives an RDFS resource describing properties and classes in the openskos namespace.

NOTE: might have to be only available on the openskos.org, i.e., http://openskos.org/vocab. This should correspond with the openskos namespace URI (`xmlns:openskos="http://openskos.org/vocab#"`). However, at the moment this is `http://openskos.org/xmlns#`.
