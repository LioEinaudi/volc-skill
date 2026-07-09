# Volc CLI Commands

Use these as generic command templates for 火山引擎机器学习平台 (ML Platform / veMLP). Always confirm the installed CLI behavior with `--help`, because flags can change by CLI version.

## Top Level

```bash
volc --help
volc version
```

Common modules:

- `configure` - set CLI configuration.
- `upgrade` - update CLI.
- `ml_task` - ML Platform custom tasks and Slurm jobs.
- `ml_model` - model repository.
- `ml_image` - image migration.
- `ml_service` - online service.
- `ml_devinstance` - development machines.

Global option:

```bash
volc --log-level fatal|error|warning|notice|info|debug <command>
```

## Configure

Do not expose real AK/SK in chat. Use environment variables or an existing profile.

```bash
volc configure --help
volc configure --ak "$VOLC_AK" --sk "$VOLC_SK" --region "$VOLC_REGION" --profile "$VOLC_PROFILE" --module ml_platform set
volc configure set --help
```

In observed CLI builds, `--ak/--sk/--region/--profile/--module` are parent `configure` flags, so place them before `set`. Verify with `volc configure --help` before changing credentials.

## Development Machines

Inspect the installed command surface:

```bash
volc ml_devinstance --help
```

Common commands:

```bash
volc ml_devinstance list
volc ml_devinstance get --id <devinstance-id>
volc ml_devinstance start --id <devinstance-id>
volc ml_devinstance stop --id <devinstance-id>
volc ml_devinstance reboot --id <devinstance-id>
volc ml_devinstance worker --help
```

List filters:

```bash
volc ml_devinstance list \
  --states Pending,Deploying,Running,Stopping,Stopped,Deleting,Abnormal \
  --name <name-substring> \
  --id <devinstance-id> \
  --queueIds <queue-id-1>,<queue-id-2> \
  --limit 20 \
  --offset 0
```

Remote-development workflow:

```bash
volc ml_devinstance list --states Running,Stopped --name <name>
volc ml_devinstance start --id <devinstance-id>
volc ml_devinstance get --id <devinstance-id>
# Use the SSH endpoint/command shown by the platform or returned details.
ssh -p <port> <user>@<host>
```

Inside the development machine, submit training jobs with `volc ml_task submit` or Slurm jobs with `volc ml_task sbatch`.

## Custom Training Tasks

Inspect the installed command surface:

```bash
volc ml_task --help
```

Common subcommands:

- `submit` - submit training task.
- `sbatch`, `sbatch-cluster` - submit Slurm sbatch scripts.
- `cancel` - cancel task.
- `get` - print task details and support exporting task config.
- `logs` - print task logs.
- `list` - list non-terminal tasks.
- `top` - print task container load.
- `upload` - upload training code.
- `export` - export config or code.
- `template` - generate preset config.
- `instance list` - print task instance details.

### Generate Config

```bash
volc ml_task template --path job_conf_example.yaml --example true
volc ml_task template --path job_conf_template.yaml --example false
```

Use a generated config for repeatable jobs:

```bash
volc ml_task submit --conf job_conf_example.yaml
volc ml_task submit --conf job_conf_example.yaml --set key=value --set nested.key=value
```

`--set` only overrides config items already defined in the config file and has lower precedence than explicit flags.

### Submit Custom Job

Minimal flag-based pattern:

```bash
volc ml_task submit \
  --task_name <job-name> \
  --entrypoint "bash train.sh" \
  --user_code_path . \
  --image <image-id-or-image-url> \
  --resource_queue_name <queue-name> \
  --framework Custom \
  --priority 4
```

Common options:

- `--conf/-c <file>`: task config file.
- `--entrypoint/--entry/-e <script-or-command>`: executable or bash command.
- `--task_name/-n <name>`, `--description/-d <text>`.
- `--user_code_path/--cp <path>`: upload code; `.gitignore` applies.
- `--image/-i <image-id-or-url>`.
- `--resource_queue_name/--queue_name <name>`: preferred queue selector.
- `--resource_queue_id/-q q_xxxx`: queue ID.
- `--resource_group_id/-g <id>`: deprecated; avoid.
- `--framework/-f TensorFlowPS|PyTorchDDP|MXNet|BytePS|MPI|Custom`.
- `--preemptible`: allow preemptible resources.
- `--priority <1-9>`: default is commonly `4`; only some bands may be user-selectable.
- `--local_diff on|off`: default is commonly `on`.
- `--access_type Public|Queue|Private`.
- `--access_users user1,user2`: only meaningful with queue-level access.
- `--copy-links/-L` or `--links`: symlink handling.
- `--output/-o <format>` or `$VOLC_DEFAULT_OUTPUT`.

### Submit Slurm/Sbatch Job

Pattern:

```bash
volc ml_task sbatch \
  --job-name <job-name> \
  --partition <queue-name-or-id> \
  --nodes <n> \
  --ntasks-per-node <n> \
  --gpus "[type:]count" \
  --time 04:00:00 \
  --user_code_path . \
  --image <image-id-or-url> \
  path/to/job.sbatch
```

Important options:

- `--conf/-c <file>` or `$MLP_SLURM_TASK_CONF`.
- `--partition/-q/--resource_queue_id/--resource_queue_name <queue>`.
- `--use-default-queue` / `--use-default-partition`: submit to public queue.
- `--gpus "[type:]count"` or `$SBATCH_GPUS`.
- `--gres "name[[:type]:count]"` or `$SBATCH_GRES`.
- `--nodes/-N`, `--ntasks-per-node`, `--mem`.
- `--parsable`: print job ID only.
- `--preemptible`, `--priority`, `--local_diff on|off`.
- `--time -1|INFINITE|UNLIMITED|minutes|minutes:seconds|hours:minutes:seconds|days-hours...`.
- `--cluster-mode on|off`: verify support with `volc ml_task sbatch --help`.
- `--master-flavor`, `--worker-flavor`.
- `--copy-links/-L` or `--links`.

### Monitor and Debug Tasks

List active or selected tasks:

```bash
volc ml_task list
volc ml_task list --status Queue,Staging,Running,Killing --name <name-or-id>
volc ml_task list --status Success,Failed,Killed --limit 20 --offset 0 --output json
volc ml_task list --format Id=id,Name=name,Status=status,Queue=ResourceQueueId,Elapsed=Elapsed
```

Task detail:

```bash
volc ml_task get --id <task-id>
volc ml_task get --id <task-id> --output json
volc ml_task get --id <task-id> --format Id=id,Name=name,Status=status,Image=Image,Priority=Priority
volc ml_task get --helpformat
```

Instances:

```bash
volc ml_task instance list --id <task-id>
volc ml_task instance list --id <task-id> --format Name=Name,Pod=PodName,State=State,Flavor=FlavorId,Exit=ExitCode
```

Logs:

```bash
volc ml_task logs --task <task-id> --lines 500
volc ml_task logs --task <task-id> --instance <instance-id> -f --timestamp
volc ml_task logs --task <task-id> --list
volc ml_task logs --task <task-id> --log-file stdout\&stderr --start-time "2026-07-09 10:00:00" --end-time "2026-07-09 12:00:00"
volc ml_task logs --task <task-id> --content <filter> --reverse --lines 200
```

Container load:

```bash
volc ml_task top --task <task-id>
volc ml_task top --task <task-id> --instance <instance-id>
```

Cancel only after confirmation:

```bash
volc ml_task cancel --id <task-id>
```

### Upload and Export

Upload code without submitting:

```bash
volc ml_task upload \
  --local_code_path . \
  --region <region> \
  --tos_end_point <tos-endpoint> \
  --timeout 600 \
  --local_diff on
```

Export task config and/or code:

```bash
volc ml_task export --task <task-id>
volc ml_task export --task <task-id> --config
volc ml_task export --task <task-id> --code --tos_end_point <tos-endpoint>
```

## Output Fields

`ml_task get` and `ml_task list` commonly support:

```text
Id, JobId, Name, JobName, CreateTime, LaunchTime, Start, FinishTime, End,
Tags, EntrypointPath, Framework, ResourceQueueId, Partition, ExitCode,
Status, Description, Elapsed, NNode, Creator, CreatorUserId, Image,
Preemptible, Priority, TaskRoleSpecs
```

`ml_task instance list` commonly supports:

```text
Name, PodName, State, FlavorId, ExitCode, LaunchTime, FinishTime, DiagInfo
```
