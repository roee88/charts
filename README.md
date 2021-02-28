# Charts

This repository hosts Mesh for Data [Helm](https://helm.sh/) charts.
The charts hosted in this repository are mirrors of charts that are maintained in other repositories within the mesh-for-data organization.

## Add Helm repository

```bash
helm repo add m4d https://mesh-for-data.github.io/charts/
helm repo update
```

## Add more charts to this repository

Follow these steps to learn how to maintain a chart in a seperate repository and sync it to this charts repository on each new tagged release.


### 1. Add GitHub workflow for publishing the chart


Create a `.github/workflows/chart-publish.yaml` file.
Replace `<chart-name>` with the name of the chart and `<charts/chart-name>` with the path to the chart:

```yaml
name: Update Charts

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    name: Update Charts
    runs-on: ubuntu-latest
    steps:
      - name: Checkout main repository
        uses: actions/checkout@v2
        with:
          path: main
      - name: Checkout charts repository
        uses: actions/checkout@v2
        with:
          repository: mesh-for-data/charts
          path: charts-repo
      - run: |
          rm -rf charts-repo/charts/<chart-name>
          cp -r main/<charts/chart-name> charts-repo/charts
      - uses: tibdex/github-app-token@v1
        id: generate-token
        with:
          app_id: ${{ secrets.CHARTS_APP_ID }}
          private_key: ${{ secrets.CHARTS_APP_PRIVATE_KEY }}
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
        with:
          path: charts-repo
          signoff: true
          token: ${{ steps.generate-token.outputs.token }}
          title: 'Update charts to new release'
          commit-message: Update charts
          committer: GitHub <noreply@github.com>
          delete-branch: true
```

You can easily modify the above to support more than a single chart.

### 2. Add GitHub workflow for testing the chart

It is recommended to add a workflow for linting and testing the chart.
See examples for a [chart-testing workflow](https://gist.github.com/roee88/205f9bf17770eedd34bab36ee3179392) and a [KubeLinter workflow](https://gist.github.com/roee88/d4867f825e36d6838a3601d5afef55e7).

### 3. Create a new release

When creating a new release of your project, be sure to update the version accordingly:

1. Use [Semantic Versioning](https://semver.org/)
2. Update the `version` field in the `Chart.yaml` file of your chart if the chart has changed since the last release.
3. Create a tag/release using the `vX.Y.Z` naming convention.

A pull request is automatically created in this charts repository after a tag is created. 
