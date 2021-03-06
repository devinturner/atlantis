# Apply Requirements
[[toc]]

## Intro
Atlantis allows you to require certain conditions be satisfied **before** an `atlantis apply`
command can be run:

* [Approved](#approved) – requires pull requests to be approved by at least one user
* [Mergeable](#mergeable) – requires pull requests to be able to be merged

## What Happens If The Requirement Is Not Met?
If the requirement is not met, users will see an error if they try to run `atlantis apply`:
![Mergeable Apply Requirement](./images/apply-requirement.png)

## Supported Requirements
### Approved
The `approved` requirement will prevent applies unless the pull request is approved
by at least one person other than the author.

#### Usage
You can set the `approved` requirement by:
1. Passing the `--require-approval` flag to `atlantis server` or
1. Creating an `atlantis.yaml` file with the `apply_requirements` key:
    ```yaml
    version: 2
    projects:
    - dir: .
      apply_requirements: [approved]
     ```

#### Meaning
Each VCS provider has different rules around who can approve:
* **GitHub** – **Any user with read permissions** to the repo can approve a pull request
* **GitLab** – You [can set](https://docs.gitlab.com/ee/user/project/merge_requests/merge_request_approvals.html#editing-approvals) who is allowed to approve
* **Bitbucket Cloud (bitbucket.org)** – A user can approve their own pull request but
  Atlantis does not count that as an approval and requires an approval from at least one user that
  is not the author of the pull request

:::tip Tip
If you want to require **certain people** to approve the pull request, look at the
[mergeable](#mergeable) requirement.
:::

### Mergeable
The `mergeable` requirement will prevent applies unless a pull request is able to be merged.

#### Usage
You can set the `mergeable` requirement by:
1. Passing the `--require-mergeable` flag to `atlantis server` or
1. Creating an `atlantis.yaml` file with the `apply_requirements` key:
    ```yaml
    version: 2
    projects:
    - dir: .
      apply_requirements: [mergeable]
     ```

#### Meaning
Each VCS provider has a different concept of "mergeability":
#### GitHub
In GitHub, if you're not using [Protected Branches](https://help.github.com/articles/about-protected-branches/) then
all pull requests are mergeable unless there is a conflict.

If you set up Protected Branches then you can enforce:
* Requiring certain status checks to be passing
* Requiring certain people to have reviewed and approved the pull request
* Requiring `CODEOWNERS` to have reviewed and approved the pull request
* Requiring that the branch is up to date with `master`

See [https://help.github.com/articles/about-protected-branches/](https://help.github.com/articles/about-protected-branches/)
for more details.

::: warning
If you have the **Restrict who can push to this branch** requirement, then
the Atlantis user needs to be part of that list in order for it to consider
a pull request mergeable.
:::

#### GitLab
For GitLab, a merge request will be mergeable if it has no conflicts and if all
required approvers have approved the pull request.

We **do not** check if there are [Unresolved Discussions](https://docs.gitlab.com/ee/user/discussions/#resolvable-comments-and-discussions) because GitLab doesn't
provide that information in their API response. If you need this feature please
[open an issue](https://github.com/runatlantis/atlantis/issues/new).

#### Bitbucket.org (Bitbucket Cloud) and Bitbucket Server (Stash)
For Bitbucket, we just check if there is a conflict that is preventing a
merge. We don't check anything else because Bitbucket's API doesn't support it.

If you need a specific check, please
[open an issue](https://github.com/runatlantis/atlantis/issues/new).

## Setting Apply Requirements
As mentioned above, you can set apply requirements via flags or `atlantis.yaml`.

### Flags Override
Flags **override** any `atlantis.yaml` settings so they are equivalent to always
having that apply requirement set.

### Project-Specific Settings
If you only want some projects/repos to have apply requirements, then you must
1. Not set the `--require-approval` or `--require-mergeable` flags, since those
   will override any `atlantis.yaml` settings
1. Specify which projects have which requirements via an `atlantis.yaml` file.
   For example if I have two directories, `staging` and `production`, I might use:
   ```yaml
   version: 2
   projects:
   - dir: staging
     # By default, apply_requirements is empty so this
     # isn't strictly necessary.
     apply_requirements: []
   - dir: production
     # This requirement will only apply to the
     # production directory.
     apply_requirements: [mergeable]


### Multiple Requirements
You can set both `apply` and `mergeable` requirements.

## Who Can Apply?
Once the apply requirement is satisfied, **anyone** that can comment on the pull
request can run the actual `atlantis apply` command.

## Next Steps
* For more information on GitHub pull request reviews and approvals see: [https://help.github.com/articles/about-pull-request-reviews/](https://help.github.com/articles/about-pull-request-reviews/)
* For more information on GitLab merge request reviews and approvals (only supported on GitLab Enterprise) see: [https://docs.gitlab.com/ee/user/project/merge_requests/merge_request_approvals.html](https://docs.gitlab.com/ee/user/project/merge_requests/merge_request_approvals.html).
* For more information on Bitbucket pull request reviews and approvals see: [https://confluence.atlassian.com/bitbucket/pull-requests-and-code-review-223220593.html](https://confluence.atlassian.com/bitbucket/pull-requests-and-code-review-223220593.html)
