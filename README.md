# eco-ci-activity-checker

This is a Github Action for checking whether a workflow needs to be run based on the last workflow run and commits to a specific branch.
It is intended to be used with automated CI tests to skip unnecessary workflows runs on branches that had no recent commits.
It checks the last workflow run, its completed state, and the date of the last commit to a branch.
It writes the output into a flag 'should_run'. If this flag is set to true, you should run the workflow. If its set to false, it should be safe to skip this job run.

If the last workflow run had a complete state of anything other than success, it sets should_run == false..
If the last workflow run was a success, but the branch had no commits since that date, then it sets should_run == false.
If the last workflow run was a success, and the branch has had commits since, then it sets should_run == true.

#### Required Inputs
`repo`: 'format: {repo-owner}/{repo-name}'  
`branch`: 'branch name to check'  
`workflow-id`: The id of the workflow this action is used in. You can get the workflow-id by checking the API endpoint [`/repos/{owner}/{repo}/actions/workflows`](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28#list-repository-workflows)

These are used for api calls to get last workflow run and recent commits

#### Outputs:
should_run: set to true or false based on the logic described above.

#### Example Use

```
jobs:
  check_date:
    runs-on: ubuntu-latest
    name: Check latest commit
    outputs:
      should_run: ${{ steps.check_main_commits.outputs.recent_commit }}
    steps:
      - id: check_main_commits
        uses: green-coding-berlin/eco-ci-activity-checker@main
        with:
          repo: 'green-coding-berlin/green-metrics-tool'
          branch: 'main'
          workflow-id: 45267392

  run-tests-main:
    needs: check_date
    if: ${{ needs.check_date.outputs.should_run == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: 'main'
          submodules: 'true'
      
      - name: 'Setup, Run, and Teardown Tests'
        uses: ./.github/actions/gmt-pytest
```
