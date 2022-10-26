# Guidance for Artifact Authors

This reference provides guidance for authoring and supporting new [OCI Artifact][artifacts] types.

This guidance aims to serve three primary goals:

1. **for Artifact Authors:** guidance for authoring new artifact types,
1. **for Registry Operators and Client Implementors:** guidance for supporting well-defined artifact types in their implementations, and,
1. as OCI Artifacts are adopted, **as a Clearing House for Well-known Artifact Types:** artifact authors can submit links to documentation describing their artifact definitions, providing implementors a set of types to test against.

## Scope

This guidance relies on implementations' support for [OCI Artifacts][artifacts].
The OCI Artifacts spec is introduced in OCI image-spec v1.1.

For compatibility with implementations that don't yet support v1.1 of the spec, see [Using OCI Images](#using-oci-images) below.
In general, new artifact types should prefer to use OCI Artifacts where available.

## Defining a Unique Artifact Type

Artifact authors should define a unique artifact type, to avoid confusion with artifacts of another type among client and registry implementations.
This is done by choosing a descriptive, unique `artifactType`.

> - **`artifactType`** *string*
> 
>   This property SHOULD be used and contain the mediaType of the artifact.
>   If defined, the value MUST comply with [RFC 6838][rfc6838], including the [naming requirements in its section 4.2][rfc6838-s4.2], and MAY be registered with [IANA][iana].

Artifact authors should not use the `vnd.oci` prefix, as these are reserved for types defined in Open Container Initiative (OCI) specifications and MUST NOT be used by other specifications and extensions.

Artifact types may include version information (`v3`) and data encoding information (`+json`).
These communicate to implementations how they should interpret the data found in other fields, such as `blobs` or [`annotations`][annotations].

Artifact types and blob `mediaType`s that may change should define a version to future-proof new enhancements.

## Defining Artifact Content

The content of an artifact can be represented through a collection of `blobs`, or through data stored in `annotations` or the `subject` descriptor.
The blobs of an artifact may be independent content, collections of content, or interpreted as an ordered collection of content, depending on the artifact's specification.

### `blobs` Content

Artifact authors may use existing standard content types (for example, the [OCI Layer media type][media-types]).
Authors are encouraged to use existing formats which may benefit from existing libraries to parse those formats.
Authors may choose to describe content with standard or custom formats, such as `.json`, `.yaml`, `.xml`, etc., or store a collection of files as a `.tar` archive.
Authors can also choose to compress the contents.
The extension of the `artifactType` reflects the format of the blob and its optional compression.

Artifacts authors may use multiple `blobs` for a range of reasons:

1. Split blobs due to size:
    - Splitting an artifact into multiple blobs can enable concurrent downloading.
    - Some artifacts will benefit from sharing common blob contents, which can be stored and fetched more efficiently.
1. Split blobs for different groupings:
    - Blobs can include different data, such as source code, licenses, software bills of material (SBOMs).
      If an artifact stores blobs of different types, it's crucial that the blobs' `mediaType`s clearly reflect what type of data they represent.

When designing blob formats, consider how a registry may de-dupe blobs that share the same content across different artifacts.
Blobs may be shared across different instances of an artifact as they're persisted to a registry, or fetched by a client.

### `annotations` Content

Artifact authors may use existing [pre-defined annotation keys][annotations-pre] where appropriate, or define their new keys, following the [rules][rules] outlined in the spec.

## Using OCI Images

OCI Images have also been used historically to represent non-image content, especially before OCI Artifacts were included in the spec.

For compatibility with implementations that don't yet support OCI Artifacts, artifact authors may choose to implement storage of non-image content in an OCI registry.
In this scheme, OCI Images' `layers` map roughly to OCI Artifacts' `blobs`, where the layers' `mediaType`s may not be the standard OCI Layer media type.
OCI Images' `.config.mediaType` maps roughly to OCI Artifacts' `artifactType`.

Examples of non-image contents represented in OCI Images include:

- [Helm charts](https://helm.sh/docs/topics/registries/)
- [Crossplane packages](https://docs.upbound.io/uxp/crossplane-concepts/packages/)
- [Sigstore](https://sigstore.dev):
  - [signatures](https://github.com/sigstore/cosign/blob/main/specs/SIGNATURE_SPEC.md)
  - [SBOMs](https://github.com/sigstore/cosign/blob/main/specs/SBOM_SPEC.md)
  - [attestations](https://github.com/sigstore/cosign/blob/main/specs/ATTESTATION_SPEC.md)
- [Tekton bundles](https://tekton.dev/docs/pipelines/tekton-bundle-contracts/)
- [Singularity filesystem bundles](https://docs.sylabs.io/guides/3.1/user-guide/oci_runtime.html)

[annotations]:       https://github.com/opencontainers/image-spec/blob/v1.1/annotations.md
[annotations-pre]:   https://github.com/opencontainers/image-spec/blob/v1.1/annotations.md#pre-defined-annotation-keys
[annotations-rules]: https://github.com/opencontainers/image-spec/blob/v1.1/annotations.md#rules
[artifacts]:         https://github.com/opencontainers/image-spec/blob/v1.1/artifact.md
[descriptor]:        https://github.com/opencontainers/image-spec/blob/v1.1/descriptor.md
[iana]:              https://www.iana.org/assignments/media-types/media-types.xhtml
[image]:             https://github.com/opencontainers/image-spec/blob/v1.1/manifest.md
[media-types]:       https://github.com/opencontainers/image-spec/blob/v1.1/media-types.md
[rfc6838-s4.2]:      https://tools.ietf.org/html/rfc6838#section-4.2
[rfc6838]:           https://tools.ietf.org/html/rfc6838
