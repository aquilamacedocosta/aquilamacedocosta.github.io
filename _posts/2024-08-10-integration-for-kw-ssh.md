---
title: GSoC - Integration Testing for kw-ssh
date: 2024-07-30 10:30:00
categories:
- gsoc
- kworkflow
- integration-tests
- kw-ssh
---

`kw-ssh` is a feature in kworkflow that simplifies remote access to machines
via SSH. It allows you to execute commands or bash scripts on a remote machine
easily. Additionally, this feature supports file and directory transfer between
the local and remote machines.

# Overview of kw-ssh Testing Architecture 

![Picture](/assets/images/kw-ssh-illustration.png)

This image illustrates the structure of the integration tests for the `kw-ssh`
feature, using containers to simulate different operating system environments.
This setup involves one container acting as the test environment, within which
tests are executed across three Linux distributions: **Debian**, **Fedora**,
and **Archlinux**.

Within this test environment, there is a second container, represented in the
image as "nested" which hosts the SSH server needed for the tests. This
configuration allows for the isolation of the test environment and execution of
`kw-ssh` commands on a simulated SSH server, without affecting the local system
or other containers.

By using containers for each Linux environment and for the SSH server, we
ensure that tests are conducted in controlled environments, avoiding
contamination between tests and maintaining result consistency. This approach
allows the functionality of `kw-ssh` to be validated across different
distributions, ensuring the code performs as expected on various platforms.

# Details of the Testing Environment

Inside the container that serves as the test environment, I copy a file from
the host machine, which is the `Containerfile` responsible for generating the
container with the SSH server. This SSH container is essential for enabling
connection tests using `kw-ssh`, ensuring that the authentication and transfer
processes work correctly.

![Picture](/assets/images/containerfile_ssh.png)

# Challenges with Nested Containers

When creating containers within containers, executing commands directly from
the host machine to the nested container becomes a challenge. To address this
complexity, I developed the `container_exec_in_nested_container()` function,
which facilitates command execution within this nested environment.

![Picture](/assets/images/container_nested.png)

This function facilitates the execution of commands in a nested container
within another container, which is common in kw-ssh integration tests. It uses
the `container_exec()` function, which executes commands directly in a
container. One of the challenges when passing commands to containers is
handling special characters, such as single quotes, which can cause issues
during execution. To address this, I used the `str_escape_single_quotes()`
function, which correctly escapes these characters, ensuring that commands are
executed reliably.

# Managing Commands in Nested Containers

The `str_escape_single_quotes()` function uses the **sed** command to add
backslashes `(\)` before any single quotes found in the string, allowing
commands containing single quotes to be interpreted correctly by the shell:

![Picture](/assets/images/str_escape_single_quotes.png)

Additionally, in the `container_exec_in_nested_container()` function, I use the
special `$''` format for strings, known as "ANSI C quoting". This format allows
escape sequences such as `\'` (escaped single quotes) to be processed correctly
by the shell. The use of `$''` is essential here to ensure that commands are
interpreted correctly, even when they contain characters that would otherwise
need to be escaped. This prevents errors when running tests.

Here's an example of how this approach is implemented:


```bash
cmd+=" ${inner_container_name} /bin/bash -c $'${inner_container_command}'"
```
By using `$'`, the string passed to the container can contain special
characters without causing problems at runtime. This is especially important
when working with nested containers, where proper string handling is critical
to the success of integration tests.


# Integration Test Example with kw-ssh

![Picture](/assets/images/test_example.png)

This test example verifies the SSH connection of `kw-ssh` using the remote
connections configuration file. Typically, this file can be found at
*~/.config/kw/remote.config*. This test illustrates how the
`container_exec_in_nested_container()` function is used to manage command
execution in nested containers and how the test is conducted across different
Linux distributions.

# Conclusion

Integration tests for kw-ssh ensure that the feature works correctly across
different Linux distributions. Using containers to isolate test environments
and the SSH server allows for precise and consistent validation. The functions
developed to manage nested containers and handle special characters ensure that
commands are executed without issues.

This approach provides confidence in the functionality of `kw-ssh`, ensuring it
performs as expected in various scenarios.
