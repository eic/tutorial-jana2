---
title: "Contributing code changes to the EICrecon repository"
teaching: 10
exercises: 10
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- How do I submit code to the EICrecon repository?

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- Understand naming conventions for EICrecon.
- Submitting a Pull Request for a contribution to EICrecon.

:::::::::::::::::::::::::::::::::::::::::::::

## Repository

We use GitHub as the main code repository tool. The repositories are located:

- [EICrecon](https://github.com/eic/EICrecon) - EIC reconstruction algorithms and EIC-related code for JANA framework
- [JANA2](https://github.com/JeffersonLab/JANA2) - The core framework

If you hesitate where to file an issue or a question, then the most probably it should be done in EICrecon project. Use the [EICrecon issues](https://github.com/eic/EICrecon/issues) tracker to file bug reports and questions.

There is also [EICrecon project board](https://github.com/orgs/eic/projects/6/views/1) where one can see what issues are in work and what could be picked up.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise

- Go to the [EICrecon project board](https://github.com/orgs/eic/projects/6/views/1) and see what tickets are marked as "TODO".

::::::::::::::: solution

The project board groups issues into columns such as "TODO", "In Progress", and "Done". The "TODO"
column lists issues that are ready to be picked up. Any of these is a reasonable starting point for
a first contribution.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## Contributing workflow

- A workflow starts from creating an issue with a bug report or a feature request. It is important to create an issue even if the subject was discussed on a meeting, personally, etc.

- Then create a branch out of the issue.

  ![Create branch from issue](fig/contributing/create_branch_1.png){alt="Creating a branch from a GitHub issue, step 1"}

  ![Create branch from issue](fig/contributing/create_branch_2.png){alt="Creating a branch from a GitHub issue, step 2"}

- After you commit and push changes to the branch, create a pull request (PR). As soon as PR is created a continious integration (CI) system will run to test the project compiles and runs on EIC environment. Any further push to this branch will trigger CI rerun the tests and check if merge is ready to be done. PRs are also a good place do discuss changes and code with collaborators. So it might be reasonable to create a PR even if not all work on issue is done. In this case create a Draft PR.

  ![Create branch from issue](fig/contributing/create_branch_3.png){alt="Opening a pull request from the new branch"}

  ![Create branch from issue](fig/contributing/create_branch_4.png){alt="Filling in the pull request information"}

  To summarize:

  - Create PR
  - Fill the information
  - Use "Draft PR" if the work is not done
  - Assign a reviewer

- Before accepting the Pull Requiest code goes through a code review by one of the core developers. If you need someone particular to review your changes - select the reviewer from the menu. Otherwise one of the developers will review the code and accept the PR.

More on the EIC contribution guide is in the [Setting up your environment tutorial](https://eic.github.io/tutorial-setting-up-environment/), [video](https://youtu.be/Y0Mg24XLomY?list=PLui8F4uNCFWm3M3g3LG2cOledhI7IvTAJ)

## Coding style

One can find coding style and other contributins policies at [CONTRIBUTING.md](https://github.com/eic/EICrecon/blob/main/CONTRIBUTING.md). It is yet to be finished but one can find current decisions on coding style there

## References

- [EICrecon](https://github.com/eic/EICrecon)
- [EICrecon project board](https://github.com/orgs/eic/projects/6/views/1)
- [EICrecon issues](https://github.com/eic/EICrecon/issues)
- [JANA2](https://github.com/JeffersonLab/JANA2)
- [EIC environment - youtube](https://youtu.be/Y0Mg24XLomY?list=PLui8F4uNCFWm3M3g3LG2cOledhI7IvTAJ)
- [EIC environment - tutorial](https://eic.github.io/tutorial-setting-up-environment/)

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- Write code in a style consistent with the rest of the repository.
- Contributions should be made through the GitHub Pull Request mechanism.

:::::::::::::::::::::::::::::::::::::::::::::
