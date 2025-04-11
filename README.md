# test project for minimal reproduction of Renovate issue

## Description

This project makes use of a single dependency in its ci script. While the dependency is versioned with strict semver (`x.y.z`), gitlab allows referencing just the major and minor version (`x.y`).

Renovate is run via docker locally with the below config file and command:

```
# config.js
module.exports = {
    autodiscover: false,
}
```

```bash
docker run -it --rm \
    -e RENOVATE_TOKEN \
    -e RENOVATE_REPOSITORIES='["msaitz/test-project"]' \
    -e RENOVATE_DRY_RUN=true \
    -e LOG_LEVEL=debug \
    -v $(PWD)/config.js:/usr/src/app/config.js \
    renovate/renovate:39.238.2
```

Note: this is a minimal reproduction of the issue originally seen in a Gitlab repository. For reproduction purposes it's been added to this Github repository (renovate won't care whether it's github or gitlab as it'd just resolve the dependency).

## Current behavior

When running the above command, Renovate detects the dependency with the default `semver-coerce` and (dry run) attempts to up update it. The update is from `x.y` to `x.y.x`.

The relevant output is

```
[...]
DEBUG: packageFiles with updates (repository=msaitz/test-project, baseBranch=main)
       "config": {
         "gitlabci": [
           {
             "packageFile": ".gitlab-ci.yml",
             "deps": [
               {
                 "datasource": "gitlab-tags",
                 "depName": "gitlab-org/components/danger-review",
                 "depType": "repository",
                 "currentValue": "2.0",
                 "registryUrls": ["https://gitlab.com"],
                 "updates": [
                   {
                     "bucket": "non-major",
                     "newVersion": "2.1.0",
                     "newValue": "2.1.0",
                     "releaseTimestamp": "2025-03-28T16:31:07.000Z",
                     "newVersionAgeInDays": 16,
                     "newMajor": 2,
                     "newMinor": 1,
                     "newPatch": 0,
                     "updateType": "minor",
                     "libYears": 0.31318052384576356,
                     "branchName": "renovate/gitlab-org-components-danger-review-2.x"
                   }
                 ],
                 "packageName": "gitlab-org/components/danger-review",
                 "versioning": "semver-coerced",
                 "warnings": [],
                 "sourceUrl": "https://gitlab.com/gitlab-org/components/danger-review",
                 "registryUrl": "https://gitlab.com",
                 "currentVersion": "2.0.0",
                 "currentVersionTimestamp": "2024-12-04T09:03:26.000Z",
                 "currentVersionAgeInDays": 130,
                 "isSingleVersion": true,
                 "fixedVersion": "2.0"
               }
             ]
           }
         ]
       }
[...]
DEBUG: branches info extended (repository=msaitz/test-project)
       "branchesInformation": [
         {
           "branchName": "renovate/gitlab-org-components-danger-review-2.x",
           "prNo": null,
           "prTitle": "Update dependency gitlab-org/components/danger-review to v2.1.0",
           "upgrades": [
             {
               "datasource": "gitlab-tags",
               "depName": "gitlab-org/components/danger-review",
               "displayPending": "",
               "fixedVersion": "2.0",
               "currentVersion": "2.0.0",
               "currentValue": "2.0",
               "newValue": "2.1.0",
               "newVersion": "2.1.0",
               "packageFile": ".gitlab-ci.yml",
               "updateType": "minor",
               "packageName": "gitlab-org/components/danger-review"
             }
           ]
         }
       ]
```

As suggested in the discssion thread (linked below), I tried to set versioning to different values. I used `semver` and `docker` with:

```
module.exports = {
    [...]
    packageRules: [
        {
            matchPackageNames: ["*gitlab-org/components/danger-review"],
            versioning: "<VERSIONING>"
        }
    ]
}
```

- With `semver`, I'm getting `"skipReason": "invalid-value"`
- With `docker`, I'm getting no updates at all.


## Expected behavior

I'd expect renovate to be able to do the update fron `2.0` to `2.1` without the patch version (given that it's a valid version in gitlab CI).


## Link to the Renovate issue or Discussion

https://github.com/renovatebot/renovate/discussions/35228
