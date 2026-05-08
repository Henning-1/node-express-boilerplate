# Runbook: TechDocs build failure

Used when the TechDocs site fails to render after a doc-only commit.

## Symptoms

- `mkdocs build` exits non-zero in CI.
- The `docs-build` workflow surfaces "Config value 'plugins': Plugin
  'techdocs-core' is not installed."

## Mitigation

Pin the `mkdocs-techdocs-core` version in `requirements-docs.txt` to
the version recorded in the last green build. Re-run the workflow.

If the failure mentions a missing markdown extension, install it via
`pip install <ext>` and add the import to `mkdocs.yml` plugin list.