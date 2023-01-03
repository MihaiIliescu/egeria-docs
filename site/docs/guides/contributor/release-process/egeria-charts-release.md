<!-- SPDX-License-Identifier: CC-BY-4.0 -->
<!-- Copyright Contributors to the Egeria project 2020. -->

### Egeria charts Release process

#### Obtaining releases / artifacts for Egeria charts 

| Location | Usage |
|---|---|
| [Github Egeria charts Release :material-github:](https://github.com/odpi/egeria-charts/releases){ target=gh } | source code in `zip` and `tar.gz` formats |
| `git` | `git checkout Vx.y` to get version as-shipped (each release is tagged at the point it is shipped) |
| `git` | `git checkout Vx.y` to get version as-shipped (each release is tagged at the point it is shipped) |

??? success "1. Agree schedule"
    - Agree on appropriate dates for branching given expected duration for testing, vacation / public holidays
         - Typically, allow 2-4 days between branching and availability
         - Communicate with team on regular calls, and via #egeria-github on Slack
         - In the last week before branching discuss holding off on any big changes in main that could destabilize the codebase
??? success "2. Track remaining issues and PRs"
    - Ensure any required issues / PRs for the release have the correct milestone set
         - Move any issues / PRs not expected to make / not required for the release to a future milestone
         - Aim to branch when most issues / PRs are complete to minimize back-porting from main, but not at the expense of impacting ongoing main development
         - Agree final branch date / criteria
 
TODO

Notes:

* There is a versioned release for each chart.
* Each chart has a chart.yaml, which contains a version number. If a change is made to the chart then the number needs to change; this can be achieved by x.y.z => x.y.z+1 for example 3.5.1 => 3.5.2   
* Each chart has a values.yaml file. This contains values for the chart, including the react UI container tag version.
* Additionally, a prerelease chart can be made prior to the release by following usual semver semantics.
For example whilst testing release 5.2, use '5.2.0-prerelease.0' then '5.2.0-prerelease.1'.
These charts are not visible from the usual helm install/search commands, but they can be seen when adding the --devel flag
* The egeria charts release process is lightweight. Branches are not used. Instead we simply continually increase the version (including support for prerelease versions that are hidden).

Since the charts aggregate many components, as well as themselves being under constant development, the release cycle will typically be more often that egeria itself.
--8<-- "snippets/abbr.md"
