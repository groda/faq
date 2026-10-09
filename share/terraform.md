# terraform

`terraform` talks to the providers named in a working directory and reconciles real infrastructure with the configuration and the state file. It does not edit the cloud by hand, and it does not apply a change just because you saved a `.tf` file. `plan` only reads and prints. `apply` and `destroy` are the verbs that write. With no subcommand it prints help and exits non-zero. HashiCorp Terraform 1.x on Linux is assumed here. The official binary is the same idea on macOS. BusyBox does not include it. OpenTofu is a different binary, `tofu`, with the same common subcommands. Exit status is 0 when the command finished as requested. It is 1 when the run failed, including a declined prompt and a lock you could not take. `plan -detailed-exitcode` is the exception: 0 means no changes, 2 means changes, 1 means a real error.

Basic form: terraform plan

The subcommand comes first. Global flags such as `-chdir=dir` sit before it. A variable value that contains spaces belongs in single quotes so the shell does not split it. A resource address that contains quotes or brackets also needs quotes. `--` before a positional plan file stops a leading dash from looking like a flag. `terraform` does not expand globs for you. The shell does, before the client sees them.

## Initialize a directory
Also asked as: terraform init; download providers; init a module; .terraform directory; backend init
`terraform init` downloads the providers and modules this directory needs and connects the backend that stores state. It writes `.terraform` and `.terraform.lock.hcl`. It does not create infrastructure. A later `plan` fails if you skip it, or if the lock file and the installed plugins disagree. Re-run it after you change `required_providers` or add a module.

```sh
terraform init -input=false
```

People run `init` in the wrong directory and then plan against an empty module. The configuration Terraform reads is the current directory, unless you passed `-chdir`. `init` does not upgrade providers that are already locked. That is `init -upgrade`. A missing backend credential fails here, not at apply, when the backend block is complete enough to try.

## See what would change
Also asked as: terraform plan; preview changes; what will terraform do; plan a diff; dry run
`terraform plan` refreshes state, compares it with the configuration, and prints the actions it would take. A `+` is a create, a `-` is a destroy, a `~` is an in-place update, and `-/+` is a replace. The command does not change infrastructure. It exits 0 even when the plan is not empty, unless you asked for a detailed exit code.

```sh
terraform plan -input=false
```

People treat a green plan as "nothing happened." Nothing was applied. The change is waiting. People also read `~` as harmless. An in-place update can still restart a resource or wipe a computed field. The plan output is the list of those actions. A saved plan file is the only way to apply exactly those actions later. A second plan can differ if the cloud changed in between.

## Apply the configuration
Also asked as: terraform apply; make the changes; deploy terraform; apply for real; write the infrastructure
`terraform apply` with no plan file makes a new plan and asks you to type `yes`. That plan is not the one you looked at earlier unless you pass the saved file. Apply writes state only after the providers accept the changes. A failed apply can leave some resources changed and the state partially updated. The error is on stderr. The status is non-zero.

```sh
terraform apply -input=false
```

People run apply in a directory that has not been inited and get a provider error that looks like a cloud failure. Run `init` first. An apply without a saved plan can also pick up a variable you did not mean to change, because it plans again. Pass the plan file when the reviewed diff is the thing you want. Typing `yes` is not `-auto-approve`. A script cannot type it.

## Apply without a prompt
Also asked as: terraform apply -auto-approve; skip yes; noninteractive apply; CI apply; do not prompt
`-auto-approve` skips the `yes` prompt and applies the plan this command just made, or the plan file you passed. `-input=false` stops Terraform from asking for a missing variable. Together they are the CI form. They do not skip errors. A rejected API call still exits non-zero.

```sh
terraform apply -input=false -auto-approve
```

People hear "auto approve" and think it means "apply the plan I already reviewed." Without a plan file, this command plans again and approves that new plan. Pass the file as the operand when the reviewed plan is what must run. `-auto-approve` on an interactive mistake is how a destroy gets applied. Read the plan line before you add the flag.

## Save a plan and apply that file
Also asked as: terraform plan -out; saved plan; apply a plan file; exact plan; binary plan
`terraform plan -out=file` writes a plan file. `terraform apply file` applies that file and does not ask you to plan again. The file pins the actions, the variable values, and the provider plugins it was made with. It is binary. `terraform show file` prints it. A plan file goes stale if the configuration or the state serial changes before you apply it.

```sh
terraform plan -input=false -out=file
```

```sh
terraform apply -input=false file
```

Use the first command to review. Use the second to apply exactly that review. People pass `-out` and then run `terraform apply` with no operand. That makes a new plan. The saved file is the operand, not a flag on apply. Do not commit the plan file. It can contain secret values you passed as variables.

## Destroy infrastructure
Also asked as: terraform destroy; tear down; delete everything; terraform plan -destroy; destroy auto approve
`terraform destroy` plans the destruction of every resource in the state for this workspace and asks you to type `yes`. It does not delete the configuration files. It does not delete the state backend. `-auto-approve` skips the prompt. `terraform plan -destroy` shows the destroy plan and does not destroy anything.

```sh
terraform plan -destroy -input=false
```

```sh
terraform destroy -input=false
```

Use the plan form to read the list. Use `destroy` when those resources should go. People run destroy from the wrong workspace and remove the wrong stack. `terraform workspace show` is the check. A resource another stack still needs is still destroyed if it is in this state. Destroy follows the graph. A failure in the middle leaves the remaining resources in state.

## Pass a variable
Also asked as: terraform -var; TF_VAR; set a variable; terraform variable; override a variable
`-var 'name=value'` sets an input variable for one command. `TF_VAR_name` in the environment sets the same variable for every command in that shell. A variable with no default and no value makes plan and apply ask, or fail if `-input=false`. The value is a string unless you pass HCL with the right quotes. Maps and lists need HCL syntax inside the single quotes.

```sh
terraform plan -var 'name=pattern' -input=false
```

People export `name=value` and expect Terraform to see it. The environment name must start with `TF_VAR_`. A `-var` beats a `TF_VAR_` value, and a `terraform.tfvars` file beats neither of those. Command-line and environment win over var files. A secret on the command line is visible in the process list. Prefer the environment or a var file with restricted permissions for those.

## Load variables from a file
Also asked as: terraform -var-file; tfvars; terraform.tfvars; variable file; auto var file
`-var-file=file` loads a `.tfvars` file. Terraform also auto-loads `terraform.tfvars`, `terraform.tfvars.json`, and any file matching `*.auto.tfvars`. A named `-var-file` is not automatic. You pass it on every plan and apply. The file is HCL or JSON. It is not the same as a `.tf` file. Variables go in tfvars. Resource blocks do not.

```sh
terraform plan -var-file=file -input=false
```

People put secrets in `terraform.tfvars` and commit the file. Auto-load means every teammate applies those values. A shared file of secrets does not belong in the repository. People also edit `variables.tf` to change a value. That file declares the variable. The value belongs in a tfvars file, `-var`, or the environment. A missing file named by `-var-file` is an error. An auto-loaded file that is absent is not.

## Format the configuration
Also asked as: terraform fmt; format hcl; terraform fmt -check; rewrite tf files; fmt recursive
`terraform fmt` rewrites `.tf` files in the current directory to the canonical layout. `-recursive` walks subdirectories. `-check` does not write. It exits non-zero if a file would change, which is the CI form. `-diff` prints the diff. Fmt does not validate the configuration. A file can be formatted and still be wrong.

```sh
terraform fmt -recursive -check
```

People run `fmt` and think the configuration was planned. It only changes whitespace and argument order. It does not touch state or the cloud. A `-check` failure is a format drift, not a plan failure. Run `terraform fmt -recursive` to write the fixes, then check again. Files outside this tree are not formatted unless you name the directory.

## Validate the configuration
Also asked as: terraform validate; check syntax; is the config valid; validate json; validate before plan
`terraform validate` checks that the configuration parses and that the providers can type-check it. It does not refresh state and it does not talk to the cloud about your resources. It needs `terraform init` first, because it loads the provider schemas. `-json` prints a machine-readable result. A valid configuration can still produce a huge plan.

```sh
terraform validate
```

People skip plan because validate passed. Validate cannot see a drift in the cloud or a bad value that is only illegal at apply time. People also run validate before init and get a missing-provider error that looks like a syntax error. Init, then validate, then plan. An error names the file and the line. A warning does not fail the command.

## List what is in state
Also asked as: terraform state list; resources in state; what does terraform manage; state addresses; list state
`terraform state list` prints the addresses in the current state. One address per line. It does not print attributes. It does not talk to the cloud. A module address looks like `module.name.resource.type.name`. The list is this workspace only. An empty list means this state has no resources, not that the cloud is empty.

```sh
terraform state list
```

People grep the configuration for a name and assume it is in state. A resource that failed to create is not in the list. A resource you removed from the configuration is still in state until a destroy or a state rm. `state list` can filter by address prefix if you pass one. No argument lists everything. The backend must be readable. A lock held by someone else makes the list wait or fail.

## Show one resource from state
Also asked as: terraform state show; resource attributes; state show address; what is stored for a resource; inspect state
`terraform state show` prints the attributes Terraform has stored for one address. The address comes from `state list`. The output is not a refresh. It can be older than the cloud. Sensitive attributes are still shown here if they are in state. This command does not change state.

```sh
terraform state show -- 'pattern'
```

People pass a resource type and get an error. The argument is the full address, such as `aws_instance.name`. People also read this as the live cloud. It is the state snapshot. `terraform plan` is what compares that snapshot with the live resource. A gone resource still shows here until state is updated. An address that is not in state is an error and a non-zero status.

## Move a resource in state
Also asked as: terraform state mv; rename a resource; move state address; refactor without destroy; state mv module
`terraform state mv` changes the address of a resource already in state. Use it when you renamed a resource in configuration, or moved it into a module, and you do not want Terraform to destroy the old address and create the new one. The cloud object is not renamed by the provider. Only the state address changes. A later plan should show no destroy for that object if the move matches the configuration.

```sh
terraform state mv -- 'pattern' 'pattern'
```

People change the name in the `.tf` file and apply. Terraform treats that as a delete and a create. The move has to happen before that apply. A wrong destination address hides the resource under a name the configuration does not have, and the next plan wants to destroy it. Run `state list` and `plan` after the move. There is no undo except another `state mv`.

## Forget a resource without destroying it
Also asked as: terraform state rm; remove from state; stop managing a resource; state rm; drop a resource
`terraform state rm` deletes an address from state and does not call the provider to destroy the real object. The resource stays in the cloud, unmanaged. The next plan will want to create it again if the configuration still declares it. Remove the block, or import it elsewhere, if that create is not what you want.

```sh
terraform state rm -- 'pattern'
```

People use `state rm` when they meant `destroy`. The cloud bill does not stop. People use it to fix a stuck resource and then apply, and Terraform creates a second copy because the block is still there. Check the plan after the rm. A typo in the address is an error. A right address and a wrong intention is a silent success. The status does not tell you the cloud object is still there.

## Import an existing resource
Also asked as: terraform import; adopt a resource; import into state; existing infrastructure; import id
`terraform import` attaches an existing cloud object to an address that is already in the configuration. The address is the first argument. The provider's import id is the second. The command writes state. It does not write the `.tf` block for you. You still need a resource block that matches. A later plan shows the drift between that block and the imported object.

```sh
terraform import -- 'pattern' 'pattern'
```

People import and expect the configuration to appear. It does not. Write the block first, import, then plan and fill in the attributes the plan wants to change. A wrong import id imports the wrong object, or fails. Import is not a refresh of a resource already in state. It errors if the address is already managed. `state rm` it first only if you mean to drop the old association.

## Print outputs
Also asked as: terraform output; root outputs; terraform output -raw; output json; read an output
`terraform output` prints the root-module outputs from state. With no name it prints them all. With a name it prints one. `-raw` prints a string with no quotes, for a shell. `-json` prints JSON. Outputs come from the last apply, not from a plan you have not applied. A sensitive output is redacted unless you asked for that name.

```sh
terraform output -raw -- name
```

People `output` a value the configuration declares but never applied. It is not in state yet, and the command fails or omits it. Apply first. `-raw` on a list or map is not a useful shell word. Use `-json` for those. A captured `-raw` value can contain a newline. Quote it in the shell. Outputs are not variables. Another directory reads them with a `terraform_remote_state` data source, not with this command.

## Switch workspace
Also asked as: terraform workspace list; terraform workspace select; terraform workspace new; workspace show; separate state
`terraform workspace list` prints workspaces. The star is the current one. `workspace select` switches. `workspace new` creates one and selects it. Each workspace has its own state in the same backend. The configuration is the same files. A plan after a switch is a plan against the other state. `workspace show` prints the name.

```sh
terraform workspace select -- name
```

People edit a workspace and think they edited a directory. The directory did not change. The state did. Destroy in the wrong workspace is the expensive form of that mistake. `default` is the workspace you have if you never created one. A backend that does not support workspaces fails these commands. Select does not create. New does not clone the other workspace's resources.

## Change only one resource
Also asked as: terraform -target; target a resource; apply one resource; terraform target; plan one address
`-target` limits the plan to one address and to resources it depends on. You can repeat the flag. It is a scalpel for a broken apply, not a way to run half a stack forever. Terraform still prints a warning. A later untargeted plan can show changes the targeted run skipped.

```sh
terraform plan -target='pattern' -input=false
```

People `-target` every apply and the untouched resources drift until a full plan surprises them. Use it to get out of a bad state, then run a full plan. A target of a module plans the whole module, not one resource inside it, unless you name that resource. The address has to match state or configuration. A typo targets nothing useful and can still exit 0 with an empty plan.

## Replace a resource
Also asked as: terraform apply -replace; force recreate; taint; replace a resource; terraform taint
`-replace` marks one address so the next plan proposes a destroy and create. You can repeat it. This is the replacement for `terraform taint`, which still exists and is worse because it writes state immediately. `-replace` belongs on the plan or apply you are running. The resource is replaced only when that plan is applied.

```sh
terraform plan -replace='pattern' -input=false
```

People taint a resource and then are surprised the next apply destroys it, including an apply someone else runs. `-replace` stays on the command you typed. A replace can fail on a resource that cannot be destroyed while something else uses it. The plan shows the dependents. Replacing is not an in-place update. Data on that resource goes away unless the provider has a separate snapshot you arranged.

## Upgrade providers
Also asked as: terraform init -upgrade; update providers; bump the lock file; newest provider; init upgrade
`terraform init -upgrade` selects the newest provider versions the constraints allow and updates `.terraform.lock.hcl`. It does not ignore the constraints in `required_providers`. A pessimistic constraint stays inside its range. The command also upgrades modules. It does not apply infrastructure. A later plan shows what the new provider wants to change.

```sh
terraform init -upgrade -input=false
```

People edit the lock file by hand. The next init puts it back, or refuses a mismatch. Change the constraint, then `-upgrade`. People `-upgrade` in CI and get a different provider than their laptop. Commit the lock file so a normal `init` installs those versions. `-upgrade` is the deliberate bump. A provider upgrade can plan a replace the old version did not. Read the plan before you apply.

## Point at a different backend
Also asked as: terraform init -reconfigure; terraform init -migrate-state; change backend; copy state to a new backend; backend config
`-reconfigure` tells init to forget the current backend settings and use the ones in the configuration now. It does not copy state. `-migrate-state` copies state from the old backend to the new one. Both are init flags. A backend block change with neither flag makes init stop and ask, or fail under `-input=false`.

```sh
terraform init -migrate-state -input=false
```

People change the backend and run a plain init. Terraform refuses because the backend changed. People use `-reconfigure` when they meant to bring the state with them, and the new backend is empty. The next plan wants to create everything. Use `-migrate-state` when the old state must move. Use `-reconfigure` when this directory should attach to a backend that already has the right state. Copy the state somewhere safe before you migrate.

## Unlock a stuck state
Also asked as: terraform force-unlock; state lock; lock id; state is locked; ConditionalCheckFailedException
`terraform force-unlock` releases a state lock when the process that held it is gone. The argument is the lock id from the error message. It does not fix a lock held by an apply that is still running. Unlocking that one lets a second apply write state over a live change. The command asks for confirmation unless `-force` is set.

```sh
terraform force-unlock -- 'pattern'
```

People force-unlock the moment a plan says it is locked. The other plan may still be running. Check the holder and the time in the error first. A stale lock from a crashed run is the case for this command. The id is not the workspace name. It is the long id in the lock error. A wrong id fails. A right id on a live apply is the dangerous success.

## Fail a pipeline when the plan has changes
Also asked as: terraform plan -detailed-exitcode; plan exit code; CI plan status; exit 2; changes pending
`-detailed-exitcode` makes `plan` exit 0 when there are no changes, 2 when there are changes, and 1 when the plan failed. A normal plan exits 0 in both the empty and the non-empty case, so a script cannot use `$?` to see drift. The flag does not apply the changes. It only changes the status.

```sh
terraform plan -input=false -detailed-exitcode
```

People write `set -e` and then treat exit 2 as a crash. It means the plan succeeded and recorded changes. A wrapper has to allow 2. People also put the flag on `apply`. It belongs on `plan`. An error is still 1, including a lock timeout and a missing variable. Those are not drift. Read the log before you call exit 2 a diff.

## Show a saved plan
Also asked as: terraform show; show plan file; terraform show json; read a plan; pretty print state
`terraform show` prints the current state in a readable form. `terraform show file` prints a saved plan file. `-json` prints the machine-readable plan or state, which is the form a policy tool wants. Show does not refresh and does not apply. A plan file shown on another machine can fail if that machine lacks the provider plugins the plan recorded.

```sh
terraform show -- file
```

People `show` a plan and think the cloud was updated. Show only renders the file. People cat the plan file and see binary. Show is the reader. `-json` output is large and includes values that may be sensitive. Do not paste it. A state show of the whole state is noisier than `state show` of one address. Use the address form when you want one resource.

## Plan against a directory that is not the current one
Also asked as: terraform -chdir; run in another directory; change directory; terraform from a monorepo; global chdir
`-chdir=dir` runs the command as if that directory were the working directory. It is a global flag, so it comes before the subcommand. Init, plan, and apply then use that directory's configuration and `.terraform`. Paths you pass for var files are still relative to the real shell directory unless you write them for the target. This is the monorepo form.

```sh
terraform -chdir=dir plan -input=false
```

People `cd` in a script and a failed `cd` plans the wrong tree. `-chdir` fails if the directory is missing, which is the safer error. A relative `-var-file` is the gotcha. From the shell's directory it may not exist inside `dir`. Pass a path the command will open, or `cd` and then use paths you can see. `-chdir` does not select a workspace. That is still `workspace select`.

## Refresh state without changing infrastructure
Also asked as: terraform apply -refresh-only; terraform plan -refresh-only; drift check; update state only; refresh only
`-refresh-only` updates state from the cloud and does not propose configuration changes. A plan with the flag shows drift that would be written into state. An apply with the flag writes that drift into state and does not create or destroy resources to match the configuration. It is how you record an out-of-band change. It is not how you fix one.

```sh
terraform plan -refresh-only -input=false
```

People run refresh-only and expect the cloud to return to the configuration. The opposite happens to state. The next normal plan is quieter because state now matches the drift. If the drift was a mistake, do not apply the refresh-only plan. Apply a normal plan instead, and read it, so Terraform changes the cloud back. `-refresh-only` still needs credentials. It still takes the state lock.

## Evaluate an expression
Also asked as: terraform console; repl; interpolate a value; try an expression; console locals
`terraform console` reads expressions against the current configuration and state and prints the result. It is a REPL. It does not apply. `exit` or end of file leaves it. A resource attribute is `resource.type.name.attr`. A local is `local.name`. Console needs init, and it needs state for values that come from real resources.

```sh
terraform console
```

People type a shell command at the console and get an expression error. The prompt evaluates Terraform expressions, not bash. People also use console to change a variable. It cannot. It only reads. A value that is unknown until apply prints a warning or an unknown, not the future id. Console is the place to test a `for` expression before you put it in a resource. It is not a second apply path.

## Turn on logs when a run fails
Also asked as: TF_LOG; terraform debug; trace logs; provider log; why did the provider fail
`TF_LOG=debug` makes this process print debug logs. `TF_LOG=trace` is louder and can include request bodies. `TF_LOG_PATH` sends the log to a file instead of stderr. The variables apply to the terraform process and to providers it starts. They do not change the plan. Unset them after the failure. A trace log often contains credentials and tokens.

```sh
TF_LOG=debug terraform plan -input=false
```

People turn on trace for a normal apply and fill the disk, or paste the log into a ticket. Debug is enough for most provider errors. The last error Terraform prints is still the summary. The log is how you see the API call under it. A crash of the CLI is not fixed by `TF_LOG`. A provider error is. `TF_LOG` set in the environment stays set for the next command in that shell.
