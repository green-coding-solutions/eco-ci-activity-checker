# green-ci-activity-checker

This is a Github Action for checking whether a branch has had commits to it in the last `time-period`  
It can also take a workflow-id to check whether it has been run in the last `time-period`  
It writes the output of these checks into two flags: `recent_commit` and `recent_workflow_run`. They are set to true if a commit was made, or a workflow run, in that `time-period`

#### Required Inputs
`repo`: 'format: {repo-owner}/{repo-name}'  
`branch`: 'branch name to check'  

These are used for an api call to [`/repos/{owner}/{repo}/commits`](https://docs.github.com/en/rest/commits/commits?apiVersion=2022-11-28#list-commits)

#### Optional Inputs
`time-unit`: Default: "day"  
`time-value`: Default: "1"  

These can be used to define how far back you want to look. must be usable as a Unix date call
`date --date '`time-value` `time-unit` ago'`

`workflow-id`: Can be used to check if this workflow was run in the last `time-period`.  
You can get the workflow-id by checking the API endpoint [`/repos/{owner}/{repo}/actions/workflows`](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28#list-repository-workflows)

#### Outputs:
recent_commit: true if a commit was made to the branch in the last `time-period`  
recent_workflow_run: true if the workflow with the `id` was run in the last `time-period`, regardless of success
