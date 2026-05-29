### What's changed in v0.1.1

* fix: kebab-case map keys for composed Object names (RFC 1123) (by @patrickleet)

  Composed Object names derived from the functions map key (e.g. 'drc-autoReady')
  were invalid k8s names — camelCase keys violate RFC 1123. Compute a kebab-cased
  objectKey in state-init and use it for Object names + the observed lookup.
  make render/validate/KCL pass admission-free, so this only surfaced on a live
  apply (ComposeResources: invalid name). Verified live via the core cutover.

  Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>


See full diff: [v0.1.0...v0.1.1](https://github.com/hops-ops/crossplane-functions-stack/compare/v0.1.0...v0.1.1)
