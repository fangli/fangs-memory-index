# Local Source Specs

This folder is for local ingestion source specs.

Source specs here are mutable local configuration. Agents may add, update, disable, or delete specs only by following `system/system/runbooks/add-source.md`.

By default, local `*.md` source specs in this folder are ignored by git because they often contain user-specific paths, accounts, or workflow details.

To create a new local source spec:

```bash
cp ../../system/examples/source-spec-template.md ./my-source.md
```

Then edit the copied file for the local runtime.
