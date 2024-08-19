<!--
SPDX-FileCopyrightText: Alliander N.V.

SPDX-License-Identifier: Apache-2.0
-->

# OSPO Code Scanner

This repository contains a GitHub Actions workflow and related configuration files to check if code repositories follow typical open source practices.
The OSPO Code Scanner helps analyze a project before it's made open source.
This is a typical responsibility of Open Source Program Offices (OSPOs).
By running various checks, the OSPO Code Scanner aims to find issues that might be important to address before the code is open sourced.

The workflow has separate steps for the different checks.
All checks are preceded by a magnifying glass symbol (🔍) to distinguish them from steps executing  technical requirements.
When issues are found, the step is marked as failed but the workflow continues running the other checks.

The OSPO Code Scanner currently assumes the repository to scan is a GitHub repository, so that the OSSF Scorecard check can use GitHub features to provide more meaningful output.

You are welcome to use the OSPO Code Scanner, adapt it to your needs, and suggest improvements through pull requests.
The configurations can and should be changed to fit your internal processes.

Despite being a GitHub Actions workflow, it is not designed to be included in a repository to check changes like pull requests.
It is designed to check an external repository.
The same tools and configurations can be used inside a repository, but then the workflow can and should be optimized for that use case.
The OSPO Code Scanner purposely runs certain tools using custom commands, whereas pull request checks can use readily available GitHub Actions for these tools.

## Checks

The workflow runs multiple checks to discover as many of the issues that might need to be addressed before releasing the code as open source software.

### Copyright and license annotations ([REUSE](https://reuse.software/))

Checks the repository with the [reuse-tool](https://github.com/fsfe/reuse-tool).
Check if the repository contains per-file annotations of copyright and license, according to the REUSE best-practices.
It also checks if the repository contains the license texts of the licenses used in the project.

The check fails if irregularities are found.

The output shows the issues and can suggest ways to improve it.

### Secrets ([TruffleHog OSS](https://github.com/trufflesecurity/trufflehog))

Looks for secrets like passwords and API-keys in the current code, including git history and commit messages.

The check fails if secrets are found.

The output shows the locations of the secrets.

### Security practices ([OSSF Scorecard](https://securityscorecards.dev/))

Runs a multitude of [security related checks](https://github.com/ossf/scorecard/blob/main/docs/checks.md) in multiple categories and scores the results.
The scorecard initiative is a mature initiative governed by the [OpenSSF](https://openssf.org/).

The check fails if there are some improvements left and the score is not a 10 out of 10.

The output shows the checks per categories and states the issues that should be addressed to increase the score.

### Repository structure ([Repolinter](https://github.com/todogroup/repolinter))

Checks contents of the repository according to a set of rules.
It mainly checks open source project best practices.
The rules are defined in [config/repolinter-rules.yaml](config/repolinter-rules.yaml).
This ruleset is inspired by [rulesets included in the Repolinter repository](https://github.com/todogroup/repolinter/tree/main/rulesets).

Some rules trigger warnings, some trigger errors.

Note that the repolinter only checks the files in the repository and doesn't check inherited project information from files that are defined in the `.github` repository of an organization.

The check fails if there are errors.

The output shows a summary of executed checks and provides some more details for checks that triggered warnings or errors.

### Non-inclusive language ([Woke](https://docs.getwoke.tech/))

Looks for words that can be considered non-inclusive and suggests alternatives.
The standard set of words is limited and so it comes with a larger set of words to check, based on the [woke-config-builder](https://gitlab.com/nicorikken/woke-config-builder/).
This large set is located in [config/woke.yaml](config/woke.yaml).

This setup was presented in a lighting talk at the Open Source Summit 2023.

[![Watch the video](https://img.youtube.com/vi/ce66oDyXQ2g/hqdefault.jpg)](https://www.youtube.com/watch?v=ce66oDyXQ2g)

Note that the writing style check using Vale has some overlap with the Woke configuration in this repository.
The Woke configuration covers more cases, which is why it still remains as a check.

The check fails if there are errors or warnings.

The output shows where the words were detected, explains why it is an issue, and suggests alternatives.

### Writing style ([Vale](https://vale.sh/))

Vale is a more comprehensive suite compared to Woke as it can check for spelling, grammar and writing style.
The configuration is located in [config/vale.ini](config/vale.ini).

It is configured to only check text-heavy files like Markdown but also Jupyter notebooks, and to ignore license text and the checkout configuration files.

It uses two packages to implement the rules:

- [RedHat package](https://vale.sh/hub/redhat/) that implements the Red Hat style guide
- [alex](https://vale.sh/hub/alex) that checks for insensitive and inconsiderate writing

Note that alex has some overlap with the Woke configuration in this repository.
The Woke configuration covers more cases, which is why it still remains as a check.

The check fails if there are warnings or errors.

The output shows all suggestions, warnings and errors.
It highlights where the issue was detected, why it is an issue and it might suggest improvements.

### Open source license policy ([OSS Review Toolkit](https://oss-review-toolkit.org/ort/))

OSS Review Toolkit (or 'ORT') is capable of analyzing dependencies of most package managers.
It can do multiple analysis, like checking against a license policy or checking for security issues.
In this setup it analyses the dependencies and evaluates them against an ORT configuration repository.
It uses the default [ort-config](https://github.com/oss-review-toolkit/ort-config) configuration by default but can be changed to use a custom ORT configuration.

The ORT check step always succeeds, regardless of the results.

As output it generates multiple reports, including a web report.
The reports can be downloaded as a .zip file on the main page of the GitHub Actions workflow run.
The web report is a single HTML file containing all necessary browser code and should be opened with a web browser.

## Setup

1. Clone this repository to your own GitHub organization.
2. Enable GitHub Actions.
3. Create a developer token with `repo:read` permissions and set it as an [Actions Repository secret](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository) `GH_REPOSITORY_READ_PERMISSIONS`. This token is used to checkout the repository to be analyzed and the ORT config repository.

## Use

The scanner can be run from the Actions tab.

![Screenshot of the 'Run workflow' dialog box on the GitHub Actions page. It contains 3 options. First to select the branch from which to run the workflow. Second the GitHub repository path, like myorg slash myrepo. It is pre-filled with PowerGridModel/power-grid-model. Third the optional Git reference or branch to scan, like main or master.](img/run.png)

## Scanner configuration

The scanner is configured from a number of sources, mostly in the [config](config/) folder.

- Some [ORT Config](https://github.com/oss-review-toolkit/ort-config) with rules and curations for OSS Review Toolkit. The link to the OSS Review Toolkit configuration is hardcoded in the workflow.
- [Repolinter rules.yaml](config/repolinter-rules.yaml) file describing the rules against which the repolinter checks.
- [vale.ini](config/vale.ini) describing the scanning configuration of Vale and the packages to be used.
- [woke.yaml](config/woke.yaml) with a long list of checks for non-inclusive language to be used by the Woke tool.

## Shortcomings and known issues

### GitHub only repository scans

It currently assumes the repository to scan is a GitHub repository, so that the OSSF Scorecard check can use GitHub features to provide more meaningful output.
Ideally it would be able to deal with other repository locations too.

### Complicated result output

All output is printed in the console, which is not user-friendly.
It would be better if the output were summarized on the Actions workflow run page as well.
Also the output of OSS Review Toolkit is not available in the same way as the other output, as it is only available as a .zip download.

### Missing organization-level configuration

The Repolinter only checks files in the repository.
Policy documents can also be placed in the `.github` repository of an organization and the repository will then inherit these policies.
This inheritance is not supported by Repolinter and it would be nice if it could deal with this.

At the same time, it is considered a best practice to have such files in each repository, even if they only contain a link to the organization-level policy document.
In that way the policy information is also available for people not viewing the code through the GitHub web interface, like a local checkout on their own computer.

### Vale makes Woke less relevant

Woke is an effective tool for checking for inclusive language, but Vale has more capabilities of analyzing text and suggesting improvements.
It would make sense to transfer the Woke configuration to one or more Vale packages and remove Woke from this setup.

### GitHub Actions dependent

This project started as a GitHub Actions workflow that also used some readily available GitHub Actions.
Over time it started using less of the readily available GitHub Actions as most of them aren't a great fit for this setup where an external repository is checked.
As such this whole scanning workflow could be automated in a more generic way, independent of GitHub Actions.
For example by having a script in a container.
That would also enable people to run it on their laptop or using some other automation tool.

# License

This project is licensed under the Apache License Version 2.0 - see [LICENSE](LICENSE) for details.

## Licenses third-party libraries

This project includes third-party libraries, which are licensed under their own respective Open Source licenses.
SPDX-License-Identifier headers are used to show which license is applicable.
The concerning license files can be found in the LICENSES directory.

# Contributing

Please read [CODE_OF_CONDUCT](CODE_OF_CONDUCT.md) and [CONTRIBUTING](CONTRIBUTING.md) for details on the process for submitting pull requests to us.

# Contact

Please read [SUPPORT](SUPPORT.md) for how to connect and get into contact with the OSPO Code Scanner project.
