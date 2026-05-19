This library is a part of Jpsonic.

To download and use Jpsonic (WAR / Docker image), please visit the main Jpsonic repository:

 * 👉 [Jpsonic (Github)](https://github.com/jpsonic/jpsonic)
 * 👉 [Jpsonic (Dokerhub)](https://hub.docker.com/r/jpsonic/jpsonic)
 * 👉 [Jpsonic (Wiki)](https://github.com/jpsonic/jpsonic/wiki/Requirements)

## Overview

jpsonic-dependencies is a master POM that manages Jpsonic's dependency libraries, excluding GPL.
Jpsonic performs library management as follows.

### Extremely Fast Catch-up with the Latest Libraries

Jpsonic manages libraries down to minor revisions and keeps pace almost entirely with the latest versions.

- In Airsonic, which was formerly upstream of Jpsonic, rework was experienced several times due to degradation during library updates.
- In recent years, CVE alerts have been on an upward trend. Prompt updates are necessary to deal with false positive suppression, which accounts for 99.9%, and to keep the CI pipeline running.
- If a library update with backward incompatibility is required, it should be identified as early as possible.

Ignoring these issues will eventually lead to failure. In the end, continuing to chase the latest versions before debt accumulates requires less man-hours.

### Management via Properties

Since keeping track of updates for all libraries is extremely difficult, simplification is achieved through property management.

```
mvn versions:display-property-updates
```

The targets for property management are empirical. The following tendencies are common:

- Relatively frequent CVE reports
- Risk of breaking backward compatibility during a bump
- Particularly deep functional relevance to Jpsonic

Aside from these, management is left entirely to spring-dependencies.

### Iterative Management

Prior to 115.0.0, library management was conducted within the Jpsonic project.
While harmful effects from stagnant library updates could be completely prevented and the history fully grasped, there was a drawback in that the main repository's git history became bloated.
Therefore, after 115.0.0, major library management was isolated to jpsonic-dependencies.

Previously, the following steps were taken:

- Manage the latest libraries in a hotfix branch
- Merge into the develop branch and verify operations
- Merge into master if there are no issues

After 115.0.0, it is slightly simplified by isolating jpsonic-dependencies:

- Version control of jpsonic-dependencies consists of major/minor/patch, and their respective SNAPSHOTs.
- When jpsonic-dependencies is updated, a SNAPSHOT is issued.
- Apply the SNAPSHOT to the develop branch.
- Once a SNAPSHOT is issued, even if there are subsequent library updates, "SNAPSHOT version iteration is not performed" (it is overwritten).
- By running CI on the develop branch, the active SNAPSHOT of the new version can be applied, enabling the shipment of development Docker images.

This is not fully automated; the versions plugin is used.

```
mvn versions:set -DnewVersion=115.0.3-SNAPSHOT -DgenerateBackupPoms=false && git commit -am "chore: bump version to 115.0.3-SNAPSHOT"
mvn versions:set -DnewVersion=115.1.0 -DgenerateBackupPoms=false && git commit -am "release: 115.1.0"
```

### Patch Release Intervals

There are no simple scheduled releases. The timing of patch releases is almost entirely governed by "whether it becomes a factor that stops the master branch CI."
Therefore, the release timing is practically determined by CVE false positive suppression, which accounts for 99.9%.

- Library updates for CVE false positive suppression
- Updates to add suppressions for CVE false positive suppression
- Furthermore, CVE false positives in Docker images may be suppressed at this timing.
- Even if there are updates to dependency libraries, they are not necessarily reflected in master immediately. Patches are pooled until the next CVE false positive occurs.

Empirically, for a scale like Jpsonic, patch releases occur every few weeks.
During the past few years of this operation, the patch release interval exceeded one month only once (CVE false positives are that frequent).
