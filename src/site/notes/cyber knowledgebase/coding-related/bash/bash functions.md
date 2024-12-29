---
{"dg-publish":true,"permalink":"/cyber-knowledgebase/coding-related/bash/bash-functions/"}
---


# functions 
```bash
function fn_name {
	<commands>
}

fn_name() {
	<commands>
}
```
- functions must be defined at the top of a script since the script is interpreted line by line
#### Return Values

When we start a new process, each `child process` (for example, a `function` in the executed script) returns a `return code` to the `parent process` (`bash shell` through which we executed the script) at its termination, informing it of the status of the execution. This information is used to determine whether the process ran successfully or whether specific errors occurred. Based on this information, the `parent process` can decide on further program flow.

|**Return Code**|**Description**|
|---|---|
|`1`|General errors|
|`2`|Misuse of shell builtins|
|`126`|Command invoked cannot execute|
|`127`|Command not found|
|`128`|Invalid argument to exit|
|`128+n`|Fatal error signal "`n`"|
|`130`|Script terminated by Control-C|
|`255\*`|Exit status out of range|
