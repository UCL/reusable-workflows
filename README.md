:exclamation: This repository is a work-in-progress. **Do not rely on these reusable workflows** at the moment! :exclamation:

# Reusable workflows for GitHub actions

Shared reusable workflows that can be referenced in GitHub Actions workflows in [repositories within the UCL organization](https://github.com/orgs/UCL/repositories).

See also the related [`UCL/gh-actions` repository](https://github.com/UCL/gh-actions) for an analogous repository for
composite actions.

## What is a reusable workflow?

Reusable workflows are GitHub Action workflow files which can be called from other workflow files,
optionally passing inputs or [secrets](https://docs.github.com/en/rest/reference/actions#secrets) to them to configure their behaviour. 
This allows a single workflow to be used across multiple repositories or multiple times within a single repository.
The aim is for this repository to collate reusable workflow files that implement GitHub Actions workflows that are likely 
to be useful across multiple repositories in the UCL organization.

## What is the difference between resuable workflows and composite actions?

Reusable workflows serve a similar purpose to another GitHub Actions feature [_composite actions_](https://github.blog/changelog/2021-08-25-github-actions-reduce-duplication-with-action-composition/)
which also help to avoid repetition in workflow files by factoring out actions in to a callable module. 
Composite actions and resusable workflows each have some limitations that can make one more appropriate than
the other in specific circumstances - see [this guide on the GitHub blog for more guidance](https://github.blog/2022-02-10-using-reusable-workflows-github-actions/#Reusable_workflows_vs._composite_actions).

## Using a workflow

To directly reference a reusable workflow in this repository in another repository, the latter repositoy __must__ also be under the UCL organization on GitHub.

The YAML syntax for calling a reusable workflow in this repository from the as a job in the `jobs` list of a workflow YAML file in another repository is as follows

```yaml
{job-id}:
    uses: ucl/reusable-workflows/.github/workflows/{name-of-workflow-file}@{tag-or-branch}
    with:
        {input-values-list}
    secrets:
        {secret-values-list}
```

where 

  * `{job-id}` is the unique (within the particular workflow jobs list) name to give the reusable workflow call job (see [descriptions of constraints on allowable names here](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_id));
  * `{name-of-workflow-file}` is the name of the `.yml` file in the [`.github/workflows`](.github/workflows) directory in this repository you wish to call;
  * `{tag-or-branch}` is the name of a [branch](https://github.com/UCL/resusable-workflows/branches) or [tag](https://github.com/UCL/resusable-workflows/tags) in this repository containing the version of the workflow file you wish to call, for example set to `main` to use the latest version on the `main` branch;
  * `{input-values-list}` is an optional list of the input values to pass to the workflow (see [description of syntax here](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idwith)), if no inputs are to be passed the `with` section should be omitted;
  * `{secret-values-list}` is an optional list of the secrets to pass to the workflow (see [description of syntax here](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idsecrets)), if no inputs are to be passed the `secrets` section should be omitted.
  
There optionally may be other jobs in the job list specified by the `jobs` property, and the outputs from preceding jobs may be passed as inputs to the reusable workflow job while conversely any outputs from the reusable workflow job can be passed as inputs to subsequent jobs.
        

## Adding a new workflow

To add a new reusable workflow please [open a pull request](https://github.com/UCL/resusable-workflows/compare). 
The workflow YAML file __must__ be placed in the `.github/workflows` directory. 
It should be given a descriptive name in all lowercase with hyphens as the word separator for example `an-example-reusable-workflow.yml`.

The workflow file should have the general structure

```yaml
name: {description}
on:
    workflow_call:
        inputs:
            {input-definitions-list}
        secrets:
            {secret-definitions-list}
        outputs:
            {output-definitions-list}
jobs:
    {job-list}
```

where

  * `{description}` is a short string description of the what the workflow does;
  * `{input-definitions-list}` is an optional list of inputs to the workflow (see [description of syntax here](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_callinputs)), if no inputs are required omit the `inputs` property;
  * `{secrets-definitions-list}` is an optional list of secrets to to pass to the workflow (see [description of syntax here](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_callsecrets), if no secrets are required omit the `secrets` property;
  * `{output-definitions-list}` is an optional list of outputs from the workflow (see [description of syntax here](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_calloutputs), if no outputs are required omit the `outputs` property;
  * `{jobs-list}` is the list of jobs to peform - see [description of job definition syntax here](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobs).

As well as the workflow YAML file you should also add a directory with the same name as the workflow file (without
the `.yml` extension) in the root of the repository which contains at a minimum a `README` file
that documents what the reusable workflow does, any 
[inputs or secrets that can be passed to the workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#using-inputs-and-secrets-in-a-reusable-workflow) and any [outputs from the workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#using-outputs-from-a-reusable-workflow). 
Optionally you may also include other files in this directory as well - for example a license file for the workflow.
