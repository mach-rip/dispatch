# Mach dispatch

`mach-rip/dispatch@v1` declares a routing operation for the Mach virtual GitHub Actions runner. The runner selects an available local runner or returns a GitHub-hosted fallback label.

The virtual runner integration is being implemented in [Mach PR #12](https://github.com/MarcPorciuncula/mach/pull/12). Publishing this action does not enable the runner for your repository. Your repository must first be configured with the Mach GitHub App and virtual runner service.

```yaml
jobs:
  route:
    runs-on: mach
    outputs:
      runner: ${{ steps.dispatch.outputs.runner }}
    steps:
      - id: dispatch
        uses: mach-rip/dispatch@v1
        with:
          fallback-runner: ubuntu-latest

  build:
    needs: route
    runs-on: ${{ needs.route.outputs.runner }}
    steps:
      - uses: actions/checkout@v4
      - run: pnpm test
```

`fallback-runner` defaults to `ubuntu-latest` and accepts a literal runner label. The action exposes `runner` and `is-fallback` (the string `true` or `false`). Run this action as the only step in the routing job. Mach interprets the declaration directly and does not download action code.

Conventional runners fail with `mach-rip/dispatch requires runs-on: mach`. Shell steps, other actions, containers, custom step conditions, and general expressions are unsupported in the routing job. Job output mappings may refer directly to either output of the dispatch step.

If the virtual runner process stops during routing, cancel and rerun the affected workflow in GitHub. Mach does not resume interrupted virtual runner sessions.
