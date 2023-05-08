# eco-ci-activity-checker

This is a Github Action for checking whether a workflow needs to be run based on the last workflow run and commits to a specific branch. It is intended to be used with automated CI tests to skip unnecessary workflow runs on branches that had no recent commits. It checks the last workflow run, its completed state, and the date of the last commit to a branch. It writes the output into a flag 'should_run'. If this flag is set to true, you should run the workflow. If its set to false, it should be safe to skip this job run.

- If the last workflow run had a completed state of anything other than success, it sets should_run == false  
- If the last workflow run was a success, but the branch had no commits since that date, then it sets should_run == false  
- If the last workflow run was a success, and the branch has had commits since, then it sets should_run == true

#### Optional inputs
`repo`: 'format: {repo-owner}/{repo-name}'. Defaults to: ${{ github.repository }}
`branch`: 'branch name to check'. Defaults to: ${{ github.ref_name }}
`workflow-id`: if you are checking for workflow runs other than the current workflow, you can pass in the workflow id. By default will check for the workflow this is called from
`personal-access-token:`: If used with a private repository, you need to create a personal access token and with repository read access and pass it along. This is needed in order to make the necessary API calls to check the for recent commits.

#### Outputs:
`should_run`: set to true or false based on the logic described above.

#### Example Use

``` yaml
jobs:
  check_date:
    runs-on: ubuntu-latest
    name: Check latest commit
    outputs:
      should_run: ${{ steps.check_main_commits.outputs.should_run }}
    steps:
      - id: check_main_commits
        uses: green-coding-berlin/eco-ci-activity-checker@main

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

## Note on private repos
 If you are running in a private repo, you must give your job actions read abilities for the github token. This  is because we make an api call to get your workflow_id which uses your $GITHUB_TOKEN, and it needs the correct permissions to do so. You also need to create a personal access token with read permissions for your repository and pass it in:

 ``` yaml
jobs:
  check_date:
    runs-on: ubuntu-latest
    permissions:
      actions: read
    name: Check latest commit
    outputs:
      should_run: ${{ steps.check_main_commits.outputs.should_run }}
    steps:
      - id: check_main_commits
        uses: green-coding-berlin/eco-ci-activity-checker@main
        with:
            personal-access-token: <your personal access token>

 ```  
