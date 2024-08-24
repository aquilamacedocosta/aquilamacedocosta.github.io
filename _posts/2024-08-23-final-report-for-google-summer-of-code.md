---
title: GSoC - Final Report of Google Summer of Code 2024
date: 2024-08-23 21:35:02
categories:
- gsoc
---

Well, after spending the last few months studying and contributing to
[kworkflow](https://kworkflow.org/) as part of **Google Summer of Code 2024**
under the **Linux Foundation**, it’s time to catalog all the contributions made
during this period. I can confidently say that this experience has been
extremely enriching and has significantly advanced my development skills.

# Proposal

My GSoC24 proposal focused on supporting and creating new integration tests for
**kworkflow**, which previously had only unit tests. Throughout the program, I
worked on introducing, enhancing, and solidifying these integration tests to
ensure more effective validation of the project's features.

# Challenges and Solutions

During my journey in the Google Summer of Code, I faced several challenges
while implementing and improving integration tests for kworkflow. Initially,
adapting the tests to run in Podman containers was a considerable challenge. I
resolved this by thoroughly studying the [Podman
documentation](https://docs.podman.io/en/latest/) and adjusting the test
scripts to ensure compatibility and performance within the testing framework.

Another major challenge was integrating tests for the `kw build` feature due to
the extended time required for kernel compilation. To mitigate this issue, the
tests were adapted to run on only one random distribution, saving both time and
resources. I utilized Podman containers and the
[shUnit2](https://github.com/kward/shunit2) framework to organize the tests,
along with specific scripts to monitor and validate CPU usage with the
`--cpu-scaling` option. This allowed `kw build` to be tested without the need
to compile the entire kernel

The integration tests for the `kw ssh` feature were more complex due to the use
of nested containers. Adapting the tests for this scenario was challenging, but
it also provided a valuable learning opportunity. Additionally, I had to manage
the execution of commands with special characters inside the containers. To
address this, I developed functions to ensure that these commands were
correctly interpreted by the shell.

I faced several challenges during my Google Summer of Code journey, but they
were overcome with the support of mentors who were always available to answer
my questions.

# Blogpost Series Timeline

To provide a detailed overview of the work done during the Google Summer of
Code, I have prepared a series of blog posts that explore different aspects of
the **kworkflow** project. Here is a timeline of the posts, with direct links to
each one:

1. [Accepted to Google Summer of Code](https://aquilamacedo.github.io/gsoc/kworkflow/integration%20tests/2024/05/03/got-accepted-into-gsoc/)

2. [Introduction to Integration Testing in kworkflow ](https://aquilamacedo.github.io/gsoc/kworkflow/integration-tests/2024/06/26/introduction-to-integration-testing/)

3. [Integration Testing for kw ssh](https://aquilamacedo.github.io/gsoc/kworkflow/integration-tests/kw-ssh/2024/07/30/integration-for-kw-ssh/)

4. [Integration Testing for kw build](https://aquilamacedo.github.io/gsoc/kworkflow/integration-tests/kw-build/2024/08/20/integration-for-kw-build/)


# Contributions

Throughout the project, I created several pull requests (PRs) addressing
different aspects of kworkflow. Each PR was carefully crafted to enhance
functionality, increase test coverage, and ensure code robustness. Below are
some of the most significant PRs:

| Pull Request | N° of Commits | 
| --- | --- |
| [setup: install kernel build dependencies](https://github.com/kworkflow/kworkflow/pull/1108) |4|
| [tests: integration: device_test: modify kw device integration test to run entirely in container](https://github.com/kworkflow/kworkflow/pull/1135) | 2 |
| [tests: integration: refactor kw_version_test to run entirely in container](https://github.com/kworkflow/kworkflow/pull/1113) | 1 |
| [run_tests: dedicate a container per test file for integration tests](https://github.com/kworkflow/kworkflow/pull/1130) | 2 |
| [run_tests: streamline test execution logic](https://github.com/kworkflow/kworkflow/pull/1148) | 1 |
| [tests: integration: self_update_test: add self-update test](https://github.com/kworkflow/kworkflow/pull/1055) | 3 |
| [tests: integration: deploy_test: introducing deploy tests](https://github.com/kworkflow/kworkflow/pull/1161) | 2 |
| [tests: integration: kw_ssh_test: Add integration tests for kw ssh functionality](https://github.com/kworkflow/kworkflow/pull/1116) | 5 |
| [tests: integration: build_test: add the kw build test](https://github.com/kworkflow/kworkflow/pull/1143) | 3 |

### Features in Development: Almost Ready to be Merged

The PRs listed above include features that are actively in development, such as
tests for the `kw ssh`, `kw build`, and `kw deploy` functionalities. Although
many of these PRs are already well-structured, they are still undergoing final
reviews and refinements. A significant portion of the important decisions has
already been discussed within the team, ensuring that the development remains
aligned with the project’s goals and needs.

# Next Steps

As a long-time contributor to **kworkflow**, I am committed to continuing my
contributions to the project. Here are the key areas I plan to focus on:

1. **Implementing Acceptance Tests**:
 I plan to develop acceptance tests that will validate multiple functionalities
 in sequence. These tests will ensure that the integration of various features
 works seamlesslyand meets the overall requirements of the project.

2. **Expanding Test Coverage**:
 Continuing to expand the test coverage is a priority. I will work on creating
 and refining tests for additional functionalities that have not yet been fully
 covered, ensuring a comprehensive and effective validation process.

3. **Migrating to New CI Pipeline**:
 An important next step is migrating the integration tests to a new CI pipeline
 developed with **Jenkins** by Marcelo Spessoto, a fellow Google Summer of Code
 2024 participant working on the kworkflow project. This migration will enhance
 the robustness and continuous verification of project changes. We look forward
 to integrating this new CI pipeline soon.

4. **Improving Documentation**:
 I will focus on improving the documentation related to the testing processes
 and overall project. This includes updating existing documentation to reflect
 new practices, enhancing clarity, and ensuring that all relevant information is
 accessible and useful to contributors and users alike.

# Acknowledgments

I would like to express my deep gratitude to my mentors, **David de Barros
Tadokoro**, **Rodrigo Siqueira**, **Paulo Meirelles**, and **Magali Lemes**.
Your attention, ideas, and feedback were crucial to the success of this project
and made the journey much more enriching. I sincerely appreciate your constant
support and valuable contributions :-).

Additionally, I would like to thank the **Linux Foundation** for the
opportunity to participate in Google Summer of Code 2024. It was an incredible
and transformative experience.
