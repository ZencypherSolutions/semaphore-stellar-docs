# Contribution Guidelines

Thank you for considering contributing to this project! We appreciate your time and effort in improving our work. Below are the guidelines to help you contribute effectively.

## Create an Issue

To suggest improvements or new research, you can create an issue by following these steps:

1. Check existing issues: Before creating a new issue, ensure that there isn't a similar one in the current list: [Issues](https://github.com/ZencypherSolutions/semaphore-stellar-docs/issues).
2. To create a new issue, click here: [Create a new Issue](https://github.com/ZencypherSolutions/semaphore-stellar-docs/issues/new).
3. Use an appropriate prefix in the title: Since this is a documentation and research repository, please use the following prefixes to organize contributions effectively:
   - docs: for issues related to documentation changes.
   - research: for issues that suggest or request research topics.

   Example: 
   - `docs: update README file`
   - `research: add resources about the Stellar ecosystem`
   
4. Clearly describe the improvement or research: Provide all relevant information or context to help others understand the issue.

## Apply to an issue

Please explain your background and how you plan to resolve the issue, as well as provide an estimate of the time it would take to resolve it.

## Fork the Repository

To make changes, first fork the main repository:

1. Fork the repository: On the main page of the repository, select the "Fork" button to create a copy in your account.
2. Clone your fork to your local machine using the following command:
   ```bash
   git clone https://github.com/your-username/semaphore-stellar-docs.git
   cd semaphore-stellar-docs
   ```

## Create a New Branch

For each contribution, create a new branch with a meaningful name that follows a clear convention:

- For documentation: `docs-meaningful-name`
- For research: `research-meaningful-name`

Example:
   ```bash
   git checkout -b docs-update-readme-file
   ```

## Make Necessary Changes

Make the necessary changes to the codebase.

We encourage contributors to adhere to the following best practices:

- Ensure that the content is clear and free of grammatical errors.
- Maintain consistency in tone and formatting throughout the document.
- Verify that all links work correctly and check for typos before submitting your changes.

## Commit Your Changes

Add the changes to the staging area:
   ```bash
   git add .
   ```
Commit with a meaningful message that clearly describes the change. Please use the conventional commit format to improve readability and organization of commit history. You can refer to the guidelines here: [Conventional Commits](https://www.conventionalcommits.org/).
   ```bash
   git commit -m "type(scope): description"
   ```
Examples:
- docs: for documentation changes:
   ```bash
   git commit -m "docs: update README file with installation instructions"
   ```
- feat: for adding a new feature:
   ```bash
   git commit -m "feat(user-auth): add login functionality"
   ```
- fix: for fixing a bug:
   ```bash
   git commit -m "fix(api): correct endpoint URL in API documentation"
   ```
- refactor: for code changes that neither fix a bug nor add a feature:
   ```bash
   git commit -m "refactor(utils): simplify data parsing function"
   ```
- test: for adding or updating tests:
   ```bash
   git commit -m "test(auth): add unit tests for login functionality"
   ```


## Push Your Changes

Push your changes to your GitHub repository:
   ```bash
   git push origin branch-name
   ```

## Create a Pull Request (PR)

When you are ready, submit a Pull Request for review. Follow these steps:

1. Go to your forked repository on GitHub.
2. Click on the "Compare & pull request" button.
3. Ensure your PR has a clear and descriptive title.
4. Complete the Pull Request template provided, offering a detailed explanation of the changes made.

All contributions must go through the PR review process to maintain quality and consistency. A project maintainer will review your PR and provide feedback or merge it into the main branch.