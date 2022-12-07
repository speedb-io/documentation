---
description: This page describes the process for a new feature request.
---

# Feature request process

If you have an idea for a new feature, we encourage you to first [discuss it with the Speedb community](https://github.com/speedb-io/speedb/discussions) regarding its viability and potential.&#x20;

Once you've discussed it with the community, you should prepare a proposal and submit it as a feature request. For your feature proposal, [use the feature request template](https://github.com/speedb-io/speedb/issues/new?assignees=\&labels=\&template=feature\_request.md\&title=). \
\
When working on a proposal, it's important to keep in mind that it takes time to discuss new ideas with the community, review it, and eventually implement it.&#x20;

We encourage you to discuss your idea early, before even writing your proposal.&#x20;

Once your proposal is accepted, the next step is to [submit a pull request](submit-a-pull-request.md#important-information-when-creating-a-pull-request).&#x20;

### A few things to note

* Bug fixes and mechanical improvements don't need a feature request.
* All new features and bug fixes must include unit tests, since they help to (a) document and validate concrete usage of a feature and its edge cases, and (b) guard against future breaking changes to lower the maintenance cost.
* Unit tests must pass with the changes.
* If a feature with failed tests (unrelated to the feature) is submitted, it will only be considered once the feature passes all testing.
* Code changes should be made with API compatibility and evolvability in mind.

Pull requests should only be sent once a proposal has been discussed, submitted, and reviewed.

It's important to note that all new features and substantive changes to Speedb need to go through a formal feature request process.&#x20;
