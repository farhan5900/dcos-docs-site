---
layout: layout.pug
navigationTitle:  Examples
title: Job Examples
menuWeight: 20
excerpt: Examples of common usage scenarios for jobs.
render: mustache
model: /1.14/data.yml
beta: true
enterprise: true
---

These examples provide common usage scenarios for jobs.

**Prerequisite:**

- [DC/OS](/1.14/installing/) and the [DC/OS CLI](/1.14/cli/install/) installed.

<a name="create-job"></a>

# Creating a Simple Job

This JSON file creates a simple job with no schedule.

1. Create a JSON file with the following contents.
    ```json
    {
      "id": "my-job",
      "description": "A job that sleeps",
      "run": {
        "cmd": "sleep 1000",
        "cpus": 0.01,
        "mem": 32,
        "disk": 0
      }
    }
    ```

1. Add the job from the DC/OS CLI.
    ```bash
    dcos job add <my-job>.json
    ```

    Alternatively, add the job using the API.
    ```bash
    curl -X POST -H "Content-Type: application/json" -H "Authorization: token=$(dcos config show core.dcos_acs_token)" $(dcos config show core.dcos_url)/service/metronome/v1/jobs -d@/Users/<your-username>/<myjob>.json
    ```

<a name="create-job-schedule"></a>

# Creating a Job with a Schedule

<p class="message--note"><strong>NOTE: </strong>This example JSON only works when you add the job from the DC/OS CLI or the UI. </p>

1. Create a JSON file with the following contents.

    ```json
    {
        "id": "my-scheduled-job",
        "description": "A job that sleeps on a schedule",
        "run": {
            "cmd": "sleep 20000",
            "cpus": 0.01,
            "mem": 32,
            "disk": 0
        },
        "schedules": [
            {
                "id": "sleep-nightly",
                "enabled": true,
                "cron": "20 0 * * *",
                "concurrencyPolicy": "ALLOW"
            }
        ]
    }
    ```

1. Add the job.
    ```bash
    dcos job add <my-scheduled-job>.json
    ```

<a name="schedule-with-api"></a>

# Creating a Job and Associating a Schedule using the API

1. Add a job without a schedule using the [instructions above](#create-job).

1. Create a JSON file with the following contents. This is the schedule for your job.

    ```json
    {
        "concurrencyPolicy": "ALLOW",
        "cron": "20 0 * * *",
        "enabled": true,
        "id": "nightly",
        "nextRunAt": "2016-07-26T00:20:00.000+0000",
        "startingDeadlineSeconds": 900,
        "timezone": "UTC"
    }
    ```

1. Add the schedule and associate it with the job.

    Via the DC/OS CLI:

    ```bash
    dcos job schedule add <job-id> <schedule-file>.json
    ```

    Via the API:

    ```bash
    curl -X POST -H "Content-Type: application/json" -H "Authorization: token=$(dcos config show core.dcos_acs_token)" $(dcos config show core.dcos_url)/service/metronome/v1/jobs/<job-id>/schedules -d@/Users/<your-username>/<schedule-file>.json
    ```

You can associate a schedule with more than one job.

<a name="partitioned-jobs"></a>

# Creating a Partitioned Jobs Environment

In this example, a partitioned jobs environment is created with the DC/OS UI. This allows you to restrict user access per job, or per job group. The jobs are created in a jobs group named `batch`, which is a child of a jobs group named `dev`.

```
├── dev
    ├── batch
        ├── job1
        ├── job2
```

The jobs groups are then assigned permissions to users `Cory` and `Alice` to restrict access.                 

**Prerequisites:**

- You must be logged in as a `superuser`.

1. Log into the DC/OS UI as a user with the `superuser` permission.

   ![Login](/1.14/img/LOGIN-EE-Modal_View-1_12.png)

   Figure 1. DC/OS Enterprise login

1.  Create the partitioned jobs.

    1.  Select **Jobs** and click **CREATE A JOB**.
    1.  In the **ID** field, type `dev.batch.job1`.
    1.  In the **Command** field, type `sleep 1000` (or another valid shell command) and click **CREATE A JOB**.

        ![Create job](/1.14/img/GUI-Jobs-New_Job_Modal_w_devbatchjob-1_12.png)

        Figure 2. New job screen

        This creates a job in this directory structure in DC/OS: **Jobs > dev > batch > job1**.

    1.  Click the **+** icon in the top right corner to create another job.

        ![Create another job](/1.14/img/GUI-Jobs-Jobs_Table-1_12.png)

        Figure 3. Create another job

    1.  In the **ID** field, type `dev.batch.job2`.
    1.  In the **Command** field, type `sleep 1000` (or another valid shell command) and click **CREATE A JOB**. You should have two jobs:

        ![create job](/1.14/img/GUI-Jobs-Partitioned_Job_Env_Detail-1_12.png)

        Figure 4. Jobs > dev > batch screen

1.  Run the jobs.

    1.  Click **Jobs > dev > batch > job1** and click **Run Now**.

        ![Run job](/1.14/img/GUI-Jobs-Job_View-Run_Now_Menu-1_12.png)

        Figure 5. **Run now** menu

    1.  Click **Jobs > dev > batch > job2** and click **Run Now**.

1.  Assign permissions to the jobs.

    1.  Select **Organization > Users** and create new users named `Cory` and `Alice`.  

        ![Create user Cory](/1.14/img/GUI-Organization-Users-Create_User_Cory-1_12.png)

         Figure 6. Create a new user

    1.  Select the user **Cory** and grant access to `job1`.
    1.  From the **Permissions** tab, click **ADD PERMISSION** and toggle the **INSERT PERMISSION STRING** button to manually enter the permissions.

        ![Add permissions cory](/1.14/img/GUI-Organization-Users-Successful_Perms_Add_Cory_DeplJobs-1_12.png)

        Figure 7. Add permissions for user 'Cory'

    1.  Copy and paste the permissions in the **Permissions Strings** field. Specify your job group (`dev/batch`), job name (`job1`), and action (`read`). Actions can be either `create`, `read`, `update`, `delete`, or `full`. To permit more than one operation, use a comma to separate them, for example: `dcos:service:metronome:metronome:jobs:/dev.batch.job1 read,update`.

        ```bash
        dcos:adminrouter:service:metronome full
        dcos:service:metronome:metronome:jobs:dev.batch.job1 read
        dcos:adminrouter:ops:mesos full
        dcos:adminrouter:ops:slave full
        dcos:mesos:master:framework:role:* read
        dcos:mesos:master:executor:app_id:/dev/batch/job1 read
        dcos:mesos:master:task:app_id:/dev/batch/job1 read
        dcos:mesos:agent:framework:role:* read
        dcos:mesos:agent:executor:app_id:/dev/batch/job1 read
        dcos:mesos:agent:task:app_id:/dev/batch/job1 read
        dcos:mesos:agent:sandbox:app_id:/dev/batch/job1 read
        ```
    1.  Click **ADD PERMISSIONS** and then **Close**.
    1.  Repeat these steps for user **Alice**, replacing `job1` with `job2` in the permissions.

1. Log out and log back in as your new user to verify the permissions. The user should now have the designated level of access to `dev/batch/job1` and `dev/batch/job2` inside the **Jobs** tab. For example, if you log in as **Alice**, you should only see **jobs2**:

    ![Alice job view](/1.14/img/GUI-Restricted_User-Jobs_View_Alice-1_12.png)

    Figure 8. Restricted view for 'Alice'
