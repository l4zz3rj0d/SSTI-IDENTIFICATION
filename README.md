![project ssti](identification.png)

## Identifying Server-Side Template Injection (SSTI)

SSTI can be identified by injecting simple template expressions and observing how the server processes them. A common first test is an arithmetic expression like *${7*7}* or *{{7*7}}*.
Follow the green lines if the payload worked or follow red line.
If the response evaluates to 49, template execution is likely occurring.

If the expression is rendered as plain text, the input is probably not vulnerable.

Further payloads using template-specific syntax (such as comments {* *}, string operations, or function calls) help narrow down the template engine (e.g., Smarty, Jinja2, Twig).
The image above illustrates a step-by-step decision flow to detect SSTI and identify the underlying template engine.
