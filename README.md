# Diff cluster connectivity

## About
This action produces a semantic diff of the expected connectivity in a given Kubernetes cluster, as dictated by resource-defining YAML files in a GitHub repository. The action will compare the connectivity before and after commits which change the cluster's endpoints (e.g., Deployments) or its NetworkPolicies. The reported diff is particularly useful for **reviewing changes** to cluster configuration, as their effect on connectivity may be hard to figure out just by looking at textual file diffs.

An example diff output (in md format):
|query|src_ns|src_pods|dst_ns|dst_pods|connection|
|---|---|---|---|---|---|
|Added connections||||||
||[demo]|[ui]|[demo]|[query-service]|TCP 8080|
||[demo]|[cli-service]|[demo]|[ui]|All connections|
|Removed connections||||||
||[demo]|[ui]|[demo]|[query-service]|UDP 8080|
||[demo]|ip block: 0.0.0.0/0|[demo]|[query-service]|All connections|

This action is part of a wider attempt to provide [shift-left automation for generating and maintaining Kubernetes Network Policies](https://shift-left-netconfig.github.io/).

## Inputs
### old-path
(Required) The path in the GitHub Workspace where the old version was checked-out
### new-path
(Required) The path in the GitHub Workspace where the new version was checked-out
### output-format
(Optional) The format in which to output verifitaion results. Either "md" (default), "yaml" or "txt".
## Outputs
### diff-results-artifact
The name of the GitHub Action artifact containing diff results
### diff-results-file
The name of the actual file in the artifact, which contains diff results
## Usage examples
### Compare changes made in a PR to the branch base (results are stored as Action artifact)
```yaml
name: network-connectivity-diff
on:
  pull_request:

jobs:
  diff-netpols:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        path: new
    - uses: actions/checkout@v2
      with:
        ref: ${{ github.base_ref }}
        path: old
    - name: Diff versions
      uses: shift-left-netconfig/netpol-diff-gh-action@v1
      with:
        new-path: new
        old-path: old
```
### Compare changes made in a PR to the branch base and store as a PR comment
```yaml
name: network-connectivity-diff
on:
  pull_request:

jobs:
  diff-netpols:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        path: new
    - uses: actions/checkout@v2
      with:
        ref: ${{ github.base_ref }}
        path: old
    - name: Diff versions
      id: diff-versions
      uses: shift-left-netconfig/netpol-diff-gh-action@v1
      with:
        new-path: new
        old-path: old
    - uses: actions/download-artifact@v2
      with:
        name: ${{ steps.diff-versions.outputs.diff-results-artifact }}
    - name: comment PR
      run: |
        cd new
        gh pr comment  ${{ github.event.number }} -F ../${{ steps.diff-versions.outputs.diff-results-file }}
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
