# About Docker Engine - Community
Docker Engine - Community is ideal for developers and small teams looking to get started with Docker and experimenting with container-based apps. Docker Engine - Community has three types of update channels, stable, test, and nightly:

- Stable gives you latest releases for general availability.
- Test gives pre-releases that are ready for testing before general availability.
- Nightly gives you latest builds of work in progress for the next major release.

# Releases
- Releases of `Docker Engine` and `Docker Client` for general availability are versioned using dotted triples. 
- The components of this triple are `YY.mm.<patch>` where the YY.mm component is referred to as the year-month release. 
- The version numbering format is chosen to illustrate `cadence` and does not guarantee `SemVer`, but the desired date for general availability. 
- The version number may have additional information, such as beta and release candidate qualifications. Such releases are considered “pre-releases”.

The cadence of the year-month releases is every 6 months starting with the 18.09 release. The patch releases for a year-month release take place as needed to address bug fixes during its support cycle.

# Nightly builds
Nightly builds are created once per day from the master branch. The version number for nightly builds take the format:
```
0.0.0-YYYYmmddHHMMSS-abcdefabcdef
```
# Pre-releases
- In preparation for a new year-month release, a branch is created from the master branch with format YY.mm when the milestones desired by Docker for the release have achieved feature-complete. 
- Pre-releases such as betas and release candidates are conducted from their respective release branches. Patch releases and the corresponding pre-releases are performed from within the corresponding release branch.
While pre-releases are done to assist in the stabilization process, no guarantees are provided.

Binaries built for pre-releases are available in the test channel for the targeted year-month release using the naming format test-YY.mm, for example test-18.09.

# General availability
- Year-month releases are made from a release branch diverged from the master branch. 
- The branch is created with format `<year>.<month>`, for example 18.09. 
- The year-month name indicates the earliest possible calendar month to expect the release to be generally available. 
- All further patch releases are performed from that branch. For example, once v18.09.0 is released, all subsequent patch releases are built from the 18.09 branch.

# Relationship between Docker Engine - Community and Docker Engine - Enterprise code
- For a given year-month release, Docker releases both `Docker Engine - Community` and `Docker Engine - Enterprise` variants concurrently. 
- `Docker Engine - Enterprise` is a superset of the code delivered in `Docker Engine - Community`. 
- Docker maintains publicly visible repositories for the `Docker Engine - Community` code as well as private repositories for the `Docker Engine - Enterprise` code. 
- Automation (a bot) is used to keep the branches between `Docker Engine - Community` and `Docker Engine - Enterprise` in sync so as features and fixes are merged on the various branches in the `Docker Engine - Community` repositories (upstream), the corresponding `Docker Engine - Enterprise` repositories and branches are kept in sync (downstream). 
- While Docker and its partners make every effort to minimize merge conflicts between Docker Engine - Community and Docker Engine - Enterprise, occasionally they will happen, and Docker will work hard to resolve them in a timely fashion.





