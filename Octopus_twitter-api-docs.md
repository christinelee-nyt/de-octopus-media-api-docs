## Octopus media APIs how-to 

[Octopus official docs](https://de.tech.nyt.net/octopus/gettingstarted/)

### What kind of problem can Octopus help me solve?
Octopus can help ingest data from a third party API and load that data into Google Cloud Storage or BigQuery.

The tool works well for downloading data from APIs where:
* The API to scrape from is available over HTTP and supports returning data in JSON format.

* The data that I'm scraping from the API is partitioned into smaller chunks. A typical partition is time. A partition by time means that I can query the API for some segment of the data by hour, day or month. Other possible segmentations exist but this is a common one. 

---

### Marketing Analytics: Octopus Twitter pull request process
#twitter

Steps in updating Octopus Twitter API: 

- make modications in `*extract.txt` and `*load.txt` files for the API of interest

- testing API change locally with Octopus via shell command by using a destination test_table to see if changes come through properly:

Temporary rename the table name which contains the changes so we can validate the modified data in a test table destination on BigQuery. In this Twitter API case, we will change the table name `table_name: {{ entity }}_metric` to `table_name: {{ entity }}_metric_Xtine0526test`

```
{% macro bq_stats_write(entity) %}
- name: twitter_{{ entity }}_metric_all_on_twitter_bq_write
  type: gcp.bq_write
  parents:
  - name: twitter_{{ entity }}_metric_all_on_twitter_add_placement_constant
    condition: successful
  - name: twitter_{{ entity }}_metric_publisher_network_add_placement_constant
    condition: successful
  map_to_result: combined
  params:
    project_id: "nyt-octopus-{{ env }}"
    dataset: twitter
    table_name: {{ entity }}_metric
    create_table: true
    partition:
      source_type: global
      key: partition_key
    schema:
      - name: date
        bq_type: DATE
        nullable: true
        source_type: global
        key: "start_time"
      - name: id
        bq_type: STRING
        nullable: true
        source_type: result
        key: "$:id"
      - name: placement
        bq_type: STRING
        nullable: false
        source_type: result
        key: "$:placement"
      - name: impressions
        bq_type: FLOAT
        nullable: true
        source_type: result
        key: "$:metrics.impressions[0]"
        default_value: Null
      - name: clicks
        bq_type: FLOAT
        nullable: true
        source_type: result
        key: "$:metrics.clicks[0]"
        default_value: Null
```


- If changes to the Octopus pipeline is successful, the below `./octopus.sh` shell command should return a `Hail Hydra` message:

a. For Twitter ads API, first process twitter_marketing_extract.txt: 
```bash
A9627:de-octopus 211493$ ./octopus.sh --config_path=configs/twitter_marketing_extract.txt --global_params start_time=2020-02-13 end_time=2020-02-14 partition_key=20200213
```

b. then run twitter_marketing_load.txt: 
```bash
A9627:de-octopus 211493$ ./octopus.sh --config_path=configs/twitter_marketing_load.txt --global_params start_time=2019-12-09 end_time=2019-12-10 partition_key=20191209          
```

Octopus response successful:
```bash
020-05-26 18:01:33,348 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,348 | task.bq_write.BQWriteTask | No conversion function found for type BOOLEAN, so returning raw value
2020-05-26 18:01:33,349 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,349 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,349 | task.bq_write.BQWriteTask | No conversion function found for type BOOLEAN, so returning raw value
2020-05-26 18:01:33,349 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,349 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
....
function found for type BOOLEAN, so returning raw value
2020-05-26 18:01:33,351 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,351 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,351 | task.bq_write.BQWriteTask | No conversion function found for type BOOLEAN, so returning raw value
2020-05-26 18:01:33,351 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,351 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,351 | task.bq_write.BQWriteTask | No conversion function found for type BOOLEAN, so returning raw value
2020-05-26 18:01:33,352 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,352 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,352 | task.bq_write.BQWriteTask | No conversion function found for type BOOLEAN, so returning raw value
2020-05-26 18:01:33,352 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,352 | task.bq_write.BQWriteTask | No conversion function found for type TIMESTAMP, so returning raw value
2020-05-26 18:01:33,352 | task.bq_write.BQWriteTask | No conversion function found for type BOOLEAN, so returning raw value
2020-05-26 18:01:33,616 | task.bq_write.BQWriteTask | Creating a table: nyt-octopus-dev.twitter.promoted_tweets_metadata
2020-05-26 18:01:34,043 | task.bq_write.BQWriteTask | Creating a reference to the table: nyt-octopus-dev.twitter.promoted_tweets_metadata
2020-05-26 18:01:34,044 | task.bq_write.BQWriteTask | Writing total 41643 rows to nyt-octopus-dev.twitter.promoted_tweets_metadata.
2020-05-26 18:01:47,966 | task.bq_write.BQWriteTask | BQ Write job submitted. Job_id: octopus_95c957ac-7c32-4610-bd4b-bf21927d801c.
2020-05-26 18:01:49,788 | task.bq_write.BQWriteTask | BQ Write job submitted. Job_id: octopus_2d8c7966-d001-4708-a9b7-1e0f810b44ba.
2020-05-26 18:01:53,298 | task.bq_write.BQWriteTask | BQ Write job octopus_95c957ac-7c32-4610-bd4b-bf21927d801c finished successfully
2020-05-26 18:01:55,086 | task.bq_write.BQWriteTask | BQ Write job octopus_2d8c7966-d001-4708-a9b7-1e0f810b44ba finished successfully
2020-05-26 18:01:55,122 | orchestrator.Orchestrator | Workflow execution has finished.
2020-05-26 18:01:55,123 | orchestrator.Orchestrator | Shutting down...
HAIL_HYDRA

```

---

- After validating changes locally, we are ready to push changes to a dev branch (name of our choice) to prepare for a pull request to the Octopus master branch: 

``` bash

A9627:de-octopus 211493$ git pull origin master
Enter passphrase for key '/Users/211493/.ssh/id_rsa': 
From github.com:nytimes/de-octopus
 * branch            master     -> FETCH_HEAD
Updating 1903077..780c4d4
Fast-forward
 .drone.yml                                   | 113 +++++---
 CODEOWNERS                                   |   6 +
 README.md                                    |  31 +++
 configs/appsflyer_datalocker.yaml            |   2 -
 configs/dv360-delete-report.yaml             |  16 ++
 configs/dv360-extract.yaml                   | 262 ++++++++++++++++++
 configs/dv360-list-report.yaml               |  12 +
 configs/dv360-load.yaml                      | 975 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 configs/facebook_ad_creatives_extract.yaml   |  93 +++++++
 configs/facebook_ad_creatives_load.yaml      |  65 +++++
 configs/facebook_marketing_extract.txt       |  19 +-
 configs/facebook_marketing_load.txt          |  49 +++-
 configs/sa360-account-extract.yaml           | 158 +++++++++++
 configs/sa360-account-load.yaml              | 153 +++++++++++
 configs/sa360-ad-extract.yaml                | 162 +++++++++++
 configs/sa360-ad-load.yaml                   | 165 +++++++++++
 configs/sa360-adgroup-extract.yaml           | 195 +++++++++++++
 configs/sa360-adgroup-load.yaml              | 240 ++++++++++++++++
 configs/sa360-campaign-extract.yaml          | 198 +++++++++++++
 configs/sa360-campaign-load.yaml             | 189 +++++++++++++
 configs/sa360-conversion-extract.yaml        | 179 ++++++++++++
 configs/sa360-conversion-load.yaml           | 111 ++++++++
 configs/twitter_marketing_extract.txt        |  94 ++++---
 configs/twitter_marketing_load.txt           |  64 +----
 docs/ADR/000-template.md                     |  23 ++
 docs/ADR/001 OAuth2Authentication.md         |  46 ++++
 docs/ADR/002-refactor-contexts.md            |  37 +++
 docs/ADR/003-ThreadPoolExecutor.md           | 160 +++++++++++
 docs/images/Draft_PR.png                     | Bin 0 -> 346159 bytes
 docs/images/PR_number_example.png            | Bin 0 -> 327823 bytes
 docs/images/ready_for_review.png             | Bin 0 -> 127156 bytes
 octopus/authenticator.py                     | 231 ++++++++--------
 octopus/context/context.py                   | 145 ++--------
 octopus/context/context_registry.py          |   6 +-
 octopus/converter.py                         |   8 +-
 octopus/credential_store.py                  |  30 +-
 octopus/executor.py                          | 125 +++++++--
 octopus/octopus.py                           |  45 ++-
 octopus/orchestrator.py                      |  25 +-
 octopus/secret_manager.py                    |  34 ++-
 octopus/specification.py                     |  11 -
 octopus/task/base_task.py                    |  23 +-
 octopus/task/bq_delete.py                    |   4 +-
 octopus/task/bq_write.py                     |  15 +-
 octopus/task/gcs_read.py                     |   8 +-
 octopus/task/gcs_write.py                    |  68 +++--
 octopus/task/http.py                         | 124 ++++++---
 octopus/task/transformations.py              |  56 ++++
 octopus/util/bq_job_processor.py             |   1 -
 octopus/util/logging_config.py               |  17 +-
 octopus/util/logging_mixin.py                |   5 +-
 octopus/workflow/edge.py                     |   4 +-
 octopus/workflow/node.py                     |   4 +-
 octopus/workflow/workflow.py                 |  53 ++--
 pull_request_template.md                     |  10 +
 scripts/clean_gcr.sh                         |  26 ++
 scripts/start_octopus.sh                     |   2 +-
 tests/authenticator/test_oauth.py            |  21 +-
 tests/authenticator/test_token.py            |   9 +-
 tests/converter/test_type_converter.py       |  32 +--
 tests/executor/test_local_executor.py        |  48 ----
 tests/executor/test_multiprocess_executor.py |  79 ++++++
 tests/orchestrator/test_orchestrator.py      |   1 -
 tests/specification/test_specification.py    |   6 +-
 tests/task/result/test_task_result.py        |   2 +
 tests/task/test_gcs_write.py                 |   8 +-
 tests/task/test_http.py                      |  52 ++--
 tests/task/test_transformations.py           |  89 +++++-
 tests/workflow/test_edge.py                  |  21 +-
 tests/workflow/test_workflow.py              |  77 ++++++
 70 files changed, 4644 insertions(+), 698 deletions(-)
 create mode 100644 CODEOWNERS
 create mode 100644 configs/dv360-delete-report.yaml
 create mode 100644 configs/dv360-extract.yaml
 create mode 100644 configs/dv360-list-report.yaml
 create mode 100644 configs/dv360-load.yaml
 create mode 100644 configs/facebook_ad_creatives_extract.yaml
 create mode 100644 configs/facebook_ad_creatives_load.yaml
 create mode 100644 configs/sa360-account-extract.yaml
 create mode 100644 configs/sa360-account-load.yaml
 create mode 100644 configs/sa360-ad-extract.yaml
 create mode 100644 configs/sa360-ad-load.yaml
 create mode 100644 configs/sa360-adgroup-extract.yaml
 create mode 100644 configs/sa360-adgroup-load.yaml
 create mode 100644 configs/sa360-campaign-extract.yaml
 create mode 100644 configs/sa360-campaign-load.yaml
 create mode 100644 configs/sa360-conversion-extract.yaml
 create mode 100644 configs/sa360-conversion-load.yaml
 create mode 100644 docs/ADR/000-template.md
 create mode 100644 docs/ADR/001 OAuth2Authentication.md
 create mode 100644 docs/ADR/002-refactor-contexts.md
 create mode 100644 docs/ADR/003-ThreadPoolExecutor.md
 create mode 100644 docs/images/Draft_PR.png
 create mode 100644 docs/images/PR_number_example.png
 create mode 100644 docs/images/ready_for_review.png
 create mode 100644 pull_request_template.md
 create mode 100755 scripts/clean_gcr.sh
 delete mode 100644 tests/executor/test_local_executor.py
 create mode 100644 tests/executor/test_multiprocess_executor.py
 create mode 100644 tests/workflow/test_workflow.py
A9627:de-octopus 211493$ git status
On branch develop
Your branch is ahead of 'origin/develop' by 93 commits.
  (use "git push" to publish your local commits)

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	cleetest_addmetrics-tw_mkting_load-Hailhydra.txt

nothing added to commit but untracked files present (use "git add" to track)
A9627:de-octopus 211493$ git branch --list
* develop
  master
A9627:de-octopus 211493$ git checkout clee_pullreqbranch
error: pathspec 'clee_pullreqbranch' did not match any file(s) known to git
A9627:de-octopus 211493$ git checkout -b clee_pullreqbranch
Switched to a new branch 'clee_pullreqbranch'
A9627:de-octopus 211493$ git branch --list
* clee_pullreqbranch
  develop
  master
A9627:de-octopus 211493$ git add .
A9627:de-octopus 211493$ git status
On branch clee_pullreqbranch
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   cleetest_addmetrics-tw_mkting_load-Hailhydra.txt
	modified:   configs/twitter_marketing_load.txt
A9627:de-octopus 211493$ git commit -m "christine added tw metrics to metric_all_on_twitter_bq_write twitter_marketing_load.txt"
[clee_pullreqbranch 1e51b19] christine added tw metrics to metric_all_on_twitter_bq_write twitter_marketing_load.txt
 2 files changed, 348 insertions(+)
 create mode 100644 cleetest_addmetrics-tw_mkting_load-Hailhydra.txt
A9627:de-octopus 211493$ git push origin develop
Enter passphrase for key '/Users/211493/.ssh/id_rsa': 
Total 0 (delta 0), reused 0 (delta 0)
remote: error: GH006: Protected branch update failed for refs/heads/develop.
remote: error: You're not authorized to push to this branch. Visit https://help.github.com/articles/about-protected-branches/ for more information.
To github.com:nytimes/de-octopus.git
 ! [remote rejected] develop -> develop (protected branch hook declined)
error: failed to push some refs to 'git@github.com:nytimes/de-octopus.git'
A9627:de-octopus 211493$ git push
fatal: The current branch clee_pullreqbranch has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin clee_pullreqbranch

A9627:de-octopus 211493$ git push --set-upstream origin clee_pullreqbranch
Enter passphrase for key '/Users/211493/.ssh/id_rsa': 
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 6.71 KiB | 6.71 MiB/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
remote: 
remote: Create a pull request for 'clee_pullreqbranch' on GitHub by visiting:
remote:      https://github.com/nytimes/de-octopus/pull/new/clee_pullreqbranch
remote: 
To github.com:nytimes/de-octopus.git
 * [new branch]      clee_pullreqbranch -> clee_pullreqbranch
Branch 'clee_pullreqbranch' set up to track remote branch 'clee_pullreqbranch' from 'origin'.

```

Preparing for a pull request submission that branches off of the Octopus master branch:

```bash
A9627:configs 211493$ git status
On branch twitter-clee-may
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
modified:   twitter_marketing_load.txt

no changes added to commit (use "git add" and/or "git commit -a")
A9627:configs 211493$ git add .
A9627:configs 211493$ git commit -m "christine added tw metrics to metric_all_on_twitter_bq_write twitter_marketing_load.txt" 
[twitter-clee-may 2074b96] christine added tw metrics to metric_all_on_twitter_bq_write twitter_marketing_load.txt
 1 file changed, 84 insertions(+)
A9627:configs 211493$ git stuats
git: 'stuats' is not a git command. See 'git --help'.

The most similar command is
status
A9627:configs 211493$ git status
On branch twitter-clee-may
nothing to commit, working tree clean


A9627:de-octopus 211493$ git push
fatal: The current branch clee_pullreqbranch has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin clee_pullreqbranch

A9627:de-octopus 211493$ git push --set-upstream origin clee_pullreqbranch
Enter passphrase for key '/Users/211493/.ssh/id_rsa': 
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 6.71 KiB | 6.71 MiB/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
remote: 
remote: Create a pull request for 'clee_pullreqbranch' on GitHub by visiting:
remote:      https://github.com/nytimes/de-octopus/pull/new/clee_pullreqbranch
remote: 
To github.com:nytimes/de-octopus.git
 * [new branch]      clee_pullreqbranch -> clee_pullreqbranch
Branch 'clee_pullreqbranch' set up to track remote branch 'clee_pullreqbranch' from 'origin'.

```

Click into the dev branch you created on Octopus github repo UI, then click on the pull request button:
![[Screen Shot 2020-05-26 at 2.45.11 PM.png]]

Submit the pull request:
![[Screen Shot 2020-05-26 at 2.45.29 PM.png]]

Pull request reviewed by DE team and ready to be merged:
![[Screen Shot 2020-05-29 at 3.42.14 PM.png]]

Success! 
![Alt Text](https://media.giphy.com/media/lkimmsqhuBjcwXUC4/giphy.gif)

---

#### (ignore git cherry-pick) Pointers from DE team on first Twitter pull request 

> Oli Simard-Morissette Today at 8:58 AM
morning @christine.lee I took a look at your PR and  the code needs to branch off of the master branch. What I suggest doing is running the following:

```bash
git checkout master
git pull
git checkout -b twitter-clee # create a new branch off of master
git cherry-pick 1e51b191f928bf294e6f888d8c46cf2ea189ea8d # get just the changes for your single commit, see the commit hash when running git log origin/clee_pullreqbranch
git push
```

> Afterwards feel free to reopen a pull request and someone on the team can take a look. I'd suggest also testing the pipeline again to make sure it works.
Make sure to not include the cleetest_addmetrics-tw_mkting_load-Hailhydra.txt log file as well (edited) 



