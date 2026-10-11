# Access Extension Specification

- **Title:** Access
- **Identifier:** <https://stac-extensions.github.io/access/v0.1.0/schema.json>
- **Field Name Prefix:** access
- **Scope:** Catalog, Collection, Item
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Access Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

This extension describes the conditions of access to data:
who can access the data and what is required to get access (e.g. registration or a fee),
whether the data can be downloaded directly or must be retrieved first, and how long the retrieval takes.

STAC describes how to authenticate ([Authentication Extension](https://github.com/stac-extensions/authentication)),
the rights to use the data (`license`) and the transaction to get data ([Order Extension](https://github.com/stac-extensions/order)),
but not the conditions of access. This extension fills this gap.
For how this extension relates to `license` and terms of service, see [Licenses and terms of use](#licenses-and-terms-of-use).

- Examples:
  - [Item example](examples/item.json): An Item with a product that is `offline` in an archive and can be ordered,
    but also `online` at a mirror and in a requester pays bucket.
    Shows the [inheritance and aggregation](#inheritance-and-aggregation) of the fields.
  - [Collection example](examples/collection.json): A Collection with data that requires a registration,
    including instructions, relation types and summaries
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [x] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections and Asset Templates)
- [x] Alternate Assets (see the [Alternate Assets Extension](https://github.com/stac-extensions/alternate-assets))
- [x] Links (incl. Link Templates)

| Field Name            | Type      | Description |
| --------------------- | --------- | ----------- |
| access:type           | string    | Whether external users can access the data. One of `open`, `embargoed`, `restricted` or `closed`. |
| access:restrictions   | \[string] | The access barriers that external users can overcome. Any of `registration`, `agreement`, `data_fee`, `transfer_fee`, `membership` or `approval`. **REQUIRED** if `access:type` is `restricted`. |
| access:instructions   | string    | How to get access to the data, e.g. registration, eligibility, restrictions or quotas. [CommonMark 0.29](https://commonmark.org/) syntax MAY be used for rich text representation. |
| access:availability   | string    | Whether the data can be downloaded directly or must be retrieved first. One of `online`, `nearline` or `offline`. |
| access:retrieval_time | string    | The expected delay until `nearline` or `offline` data is available for download, as an [ISO 8601 Duration](https://en.wikipedia.org/wiki/ISO_8601#Durations). |

In Catalogs, Collections and Item Properties, the fields describe the access to the data (i.e. the assets), not to the STAC metadata.
In Assets, Alternate Assets and Links, the fields describe the access to the resource that is linked to.

The conditions of access depend on the copy of the data and its distributor, not on the data itself.
The same data may be `online` at one distributor and `offline` at another,
or require a registration at one distributor and a transfer fee at another.
Each copy can be described as an Alternate Asset with its own fields.

### Additional Field Information

#### access:type

The values and their definitions are taken from the `dataAccessType` of the
[re3data.org Metadata Schema 4.0](https://doi.org/10.48440/re3.014):

- `open`: There are no access barriers.
- `embargoed`: External users can't overcome the access barriers until the data is released and openly accessible.
- `restricted`: External users can overcome the access barriers, see [`access:restrictions`](#accessrestrictions).
- `closed`: External users can't overcome the access barriers, e.g. only the metadata is public.

Data that requires a free registration is `restricted`, not `open`.
For example, data from the Copernicus Data Space Ecosystem is free, but requires an account
(see the [Collection example](examples/collection.json)).

The values can be mapped to the
[EU Access Right](https://op.europa.eu/en/web/eu-vocabularies/dataset/-/resource?uri=http://publications.europa.eu/resource/dataset/access-right)
vocabulary (used in DCAT-AP) and the [COAR Access Rights](https://vocabularies.coar-repositories.org/access_rights/) vocabulary:

| `access:type` | EU Access Right | COAR Access Rights   |
| ------------- | --------------- | -------------------- |
| `open`        | `PUBLIC`        | open access          |
| `embargoed`   | `NON_PUBLIC`    | embargoed access     |
| `restricted`  | `RESTRICTED`    | restricted access    |
| `closed`      | `NON_PUBLIC`    | metadata only access |

#### access:restrictions

The access barriers that external users can overcome:

- `registration`: An account must be created, e.g. a free account in a data portal.
- `agreement`: Terms or a license must be accepted explicitly before the data can be accessed, e.g. in a click-through dialog.
- `data_fee`: A fee must be paid for the data itself, e.g. for commercial data.
- `transfer_fee`: The data itself is free, but the costs for transferring the data must be paid,
  e.g. for data in a requester pays bucket or for the delivery on a physical medium.
- `membership`: Only members of specific institutions or groups can access the data.
- `approval`: Access must be requested and approved, e.g. based on a project proposal or the eligibility of the user.

The values `registration` and `membership` correspond to `registration` and `institutionalMembership`
in the `dataAccessRestriction` of the [re3data.org Metadata Schema 4.0](https://doi.org/10.48440/re3.014).
The values `data_fee` and `transfer_fee` both correspond to `feeRequired`.
The values `agreement` and `approval` are additions, which are common for Earth observation data.

The field should only be provided if `access:type` is `restricted`.
[`access:instructions`](#accessinstructions) can describe the details,
and links can point to the corresponding pages, e.g. where to register (see [Relation types](#relation-types)).

The restrictions can differ between copies of the data,
e.g. a registration at a data portal and a transfer fee for a requester pays bucket (see the [Item example](examples/item.json)).

#### access:availability

The values are defined by what a client needs to do to download the data, not by the storage technology:

- `online`: The data can be downloaded directly.
- `nearline`: The data is not available immediately, but the download request itself triggers the retrieval.
  Clients must expect a delay, but no order is needed.
- `offline`: The data must be retrieved explicitly (e.g. by placing an order) before it can be downloaded.

The values are aligned with the on-line, near-line and off-line storage of the
[Reference Model for an Open Archival Information System (OAIS)](https://ccsds.org/Pubs/650x0m3.pdf).
They are similar to the `statusSubType` (`ON-LINE` or `OFF-LINE`) of the
[OGC® Earth Observation Metadata profile of Observations & Measurements, Table 5](https://docs.ogc.org/is/10-157r4/10-157r4.html#24)
and the boolean `Online` property of the Copernicus Data Space Ecosystem OData API.

##### Implementation of nearline data

`nearline` can be implemented with the
[Asynchronous Request-Reply pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/asynchronous-request-reply):
the request for the data returns [`202 Accepted`](https://www.rfc-editor.org/rfc/rfc9110#section-15.3.3)
with a [`Retry-After`](https://www.rfc-editor.org/rfc/rfc9110#section-10.2.3) header until the data has been retrieved.
Only `GET` requests should start a retrieval, not `HEAD` requests, so that crawlers and link checkers don't trigger retrievals.

#### access:retrieval_time

The expected time between the request (or order) and the moment the data is available for download.
It is similar to the `turnaround` of the `MD_StandardOrderProcess` in ISO 19115.

It should only be provided for `nearline` and `offline` data and is ignored for `online` data, e.g. if inherited.

#### access:instructions

Describes the details of how to get access to the data, for information that a client can't process automatically, for example:

- where to register or request an approval,
- who is eligible (e.g. only research institutions or public authorities),
- which terms need to be accepted,
- prices and quotas.

### Inheritance and aggregation

The fields are inherited as defined in
[STAC](https://github.com/radiantearth/stac-spec/blob/master/commons/assets.md#additional-fields).
In addition, Alternate Assets inherit the fields of their Asset.
Assets and Alternate Assets with a different value MUST provide it, including Assets that are not data, e.g. thumbnails.
`access:restrictions` is inherited and overridden as a whole.

If the data differs in its conditions of access, the Item Properties describe the most accessible data,
so that users can search for what they can get.
The data consists of all Assets with the role `data` (or all Assets, if none has this role) and their Alternate Assets.

- `access:type`: The first applicable value of `open`, `restricted`, `embargoed` and `closed`.
- `access:restrictions`: All restrictions of the data with this `access:type`.
- `access:availability`: The first applicable value of `online`, `nearline` and `offline`.
- `access:retrieval_time`: The longest delay of all data that is not `online`, i.e. until all data is available.

The top-level fields of Catalogs and Collections describe all of their data in the same way, including their Items,
but they are not inherited by the Items.

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type             | Description |
| ---------------- | ----------- |
| registration     | The page where users can register to get access to the data, see the restriction `registration`. |
| access-request   | The page where users can request access to the data, see the restriction `approval`. |
| terms-of-service | The terms of service that apply to accessing the data, see the restriction `agreement`. Registered at [IANA][iana-rels]. |
| payment          | The page where users can pay for the data, see the restrictions `data_fee` and `transfer_fee`. Registered at [IANA][iana-rels]. |

[iana-rels]: https://www.iana.org/assignments/link-relations/link-relations.xhtml

## Licenses and terms of use

Restrictions are described in different places, depending on what they restrict:

| Restriction                    | Example                                  | Where to describe it |
| ------------------------------ | ---------------------------------------- | -------------------- |
| Use of the data                | Attribution required, not for navigation | `license` field and links with the relation type `license` |
| Agreement with the distributor | Terms of service of a data portal        | Links with the relation type `terms-of-service` (see [Relation types](#relation-types)) |
| Access to the data             | Registration, eligibility, quotas        | [`access:type`](#accesstype), [`access:restrictions`](#accessrestrictions) and [`access:instructions`](#accessinstructions) |

This follows the distinction between use constraints and access constraints in ISO 19115 (`MD_LegalConstraints`).

The `license` field contains an SPDX license identifier or expression, or `other`.
It must not be used for terms of service or access conditions.

A standard license with additional use constraints is a different license.
For example, for data under CC-BY-4.0 that must not be used for navigation,
set `license` to `other` and link to the license text with the relation type `license`.
The title of the link can name the base license, e.g. "CC-BY-4.0, not for navigation".

## Relationship with other extensions

### Authentication Extension

The [Authentication Extension](https://github.com/stac-extensions/authentication) describes
everything that a client can automate to authenticate, e.g. the protocol, endpoints, OAuth flows and scopes.
`access:type` and `access:restrictions` describe who can access the data,
and `access:instructions` describes what no protocol captures, e.g. eligibility, terms to accept, quotas or manual account approval.
`access:availability` and `access:retrieval_time` describe the state of the data, regardless of who accesses it.

For example, "create an account on portal X" belongs in `access:instructions`,
but "request a token from this endpoint and pass it as a Bearer token" belongs in `auth:schemes`.

Authentication alone doesn't make data `restricted`.
For example, data that requires a token that anybody can request anonymously is `open`.

### Order Extension

`access:availability` and [`order:status`](https://github.com/stac-extensions/order#orderstatus) describe different things:

- `access:availability` describes the state of the data, which is known before any order exists.
- `order:status` describes the transaction that changes the state, e.g. the retrieval of `offline` data.

Data that is `online` may still be orderable, e.g. for a delivery into the storage of the user.
`nearline` data doesn't need an order, so there's no `order:status` to track.

Once an order has succeeded, `access:availability` changes to `online`.
The [Item example](examples/item.json) shows an `offline` product that can be ordered.

### Storage Extension

The [Storage Extension](https://github.com/stac-extensions/storage) describes how a client accesses data in cloud storage,
e.g. whether the requester pays for the transfer of the data (`requester_pays`).
The access fields describe the conditions of access in a way that can be searched, e.g. in Collection summaries.

If an Asset or Alternate Asset references a storage scheme with `requester_pays` set to `true`,
`access:restrictions` SHOULD include `transfer_fee` for it.

### Product Extension

[`product:status`](https://github.com/stac-extensions/product#productstatus) is a property of the product itself,
e.g. whether it is `planned`, `archived` or `rejected`.
`access:availability` is a property of a copy of the data at a distributor.
Both are independent of each other.
For example, `archived` doesn't imply that the data must be retrieved before it can be downloaded.

### Other fields

- The `unpublished` field of the [Timestamps Extension](https://github.com/stac-extensions/timestamps)
  indicates that the data was withdrawn and is not accessible any longer.
  `offline` data is still accessible, but needs to be retrieved first.
- The `storage:tier` field (in v1 of the [Storage Extension](https://github.com/stac-extensions/storage))
  described the provider-specific storage class and was removed in v2.
  `access:availability` describes what a client needs to do instead.

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PRs are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:

```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:

```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:

```bash
npm run format-examples
```
