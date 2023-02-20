# `run-on-comment.yml`

Reusable workflow for triggering tests or other commands by using a specially formatted issue or pull-request comment of the form

```
/run {keyword}
```

where `{keyword}` is a specifiable keyword. Issues comments trigger a workflow run which checks out the latest commit on the default branch of the repository, while comments on a pull-request instead checkout the latest commit on the pull-request branch.

## Using the workflow

The workflow runs a list of commands specified as an input `commands` to the workflow before replying to the issue or pull-request thread with a comment indicating if the job completed successfully and some basic run information (job ID and link to run log, time in minutes to complete, SHA of commit on PR branch for which commands were run). There are also workflow inputs `runs-on` to specify the runner type to use, `keyword` to specify the triggering keyword, `description` to define a description to use in comment and `timeout-minutes` to optionally override the default timeout for jobs of 360 minutes.

For example to add a run triggered with keyword `special-test` and run on a self-hosted runner something like the following would be added to a YAML file in the `.github/workflows` directory

```YAML
name: Run extra tests on issue or pull-request comments

on:
  issue_comment:
    types:
      - created
      
jobs:
  special_test:
    uses: ucl/reusable-workflows/.github/workflows/run-on-comment.yml
    with:
      runs-on: self-hosted
      keyword: special-test
      commands: ./run-special-test
      description: Run special test
      timeout-minutes: 720
```

## Using a GitHub App to generate authentication token

By default the workflow uses the `GITHUB_TOKEN` authentication token automatically generated at the start of a workflow run to authenticate calls to the GitHub REST API. This can lead to timeout errors when running tests which exceed [the 24 hour expiry time on the `GITHUB_TOKEN` authentication token](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) when trying to create a comment with the results of the test after completion. The `run-on-comment` workflow therefore optionally allows instead using a GitHub App to generate the access token used to create the comment after the tests have completed. The App needs to have Actions (read) and Pull Requests (read and write) permissions at the repository level.

To use this feature, set the optional boolean `use-application-token` input to `true` and specify secrets `application-id` and `application-private-key` corresponding to the App ID and private key respectively. For instance the above usage example would be changed to

```YAML
name: Run extra tests on issue or pull-request comments

on:
  issue_comment:
    types:
      - created
      
jobs:
  special_test:
    uses: ucl/reusable-workflows/.github/workflows/run-on-comment.yml
    with:
      runs-on: self-hosted
      keyword: special-test
      commands: ./run-special-test
      description: Run special test
      timeout-minutes: 720
      use-application-token: true
    secrets:
      application-id: ${{ secrets.COMMENT_BOT_APP_ID }}
      application-private-key: ${{ secrets.COMMENT_BOT_APP_PRIVATE_KEY }}
```

where it is assumed that repository or organization level secrets `COMMENT_BOT_APP_ID` and `COMMENT_BOT_PRIVATE_KEY` have been set containing respectively the App ID and private key for a GitHub App with the required permission levels. If the App is installed at the organization level, then the optional `application-organization` input should be set to specify the name of the organization the App is installed for.

## Acknowledgements

This workflow was originally developed as part of the Thanzi La Onze Model (https://www.tlomodel.org/) project. The workflow here is an extension of the [idea described in this blog post](https://pakstech.com/blog/gh-actions-issue-comments/) with some extra features and checks:

  * Ensuring only users with write or admin access to repository can trigger workflow runs.
  * Optionally using a GitHub App to generate a token to make queries to the GitHub REST API after the test has run to avoid timeout errors.
  * Responding with more detailed information about workflow run.
  
The GitHub App based access tokens are created using the [`workflow-application-token-action`](https://github.com/peter-murray/workflow-application-token-action).
