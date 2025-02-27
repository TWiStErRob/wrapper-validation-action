<p align="center">
  <a href="https://github.com/gradle/wrapper-validation-action/actions"><img alt="gradle/wrapper-validation-action status" src="https://github.com/gradle/wrapper-validation-action/workflows/ci/badge.svg"></a>
</p>

# Gradle Wrapper Validation Action

This action validates the checksums of [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) JAR files present in the source tree and fails if unknown Gradle Wrapper JAR files are found.

## The Gradle Wrapper Problem in Open Source

The `gradle-wrapper.jar` is a binary blob of executable code that is checked into nearly
[2.8 Million GitHub Repositories](https://github.com/search?l=&q=filename%3Agradle-wrapper.jar&type=Code).

Searching across GitHub you can find many pull requests (PRs) with helpful titles like 'Update to Gradle xxx'.
Many of these PRs are contributed by individuals outside of the organization maintaining the project.

Many maintainers are incredibly grateful for these kinds of contributions as it takes an item off of their backlog.
We assume that most maintainers do not consider the security implications of accepting the Gradle Wrapper binary from external contributors.
There is a certain amount of blind trust open source maintainers have.
Further compounding the issue is that maintainers are most often greeted in these PRs with a diff to the `gradle-wrapper.jar` that looks like this.

![Image of a GitHub Diff of Gradle Wrapper displaying text 'Binary file not shown.'](https://user-images.githubusercontent.com/1323708/71915219-477d7780-3149-11ea-9254-90c80dbffb0a.png)

A fairly simple social engineering supply chain attack against open source would be contribute a helpful “Updated to Gradle xxx” PR that contains malicious code hidden inside this binary JAR.
A malicious `gradle-wrapper.jar` could execute, download, or install arbitrary code while otherwise behaving like a completely normal `gradle-wrapper.jar`.

## Solution

We have created a simple GitHub Action that can be applied to any GitHub repository.
This GitHub Action will do one simple task:
verify that any and all `gradle-wrapper.jar` files in the repository match the SHA-256 checksums of any of our official releases.

If any are found that do not match the SHA-256 checksums of our official releases, the action will fail.

Additionally, the action will find and SHA-256 hash all
[homoglyph](https://en.wikipedia.org/wiki/Homoglyph)
variants of files named `gradle-wrapper.jar`,
for example a file named `gradlе-wrapper.jar` (which uses a Cyrillic `е` instead of `e`).
The goal is to prevent homoglyph attacks which may be very difficult to spot in a GitHub diff.
We created an example [Homoglyph attack PR here](https://github.com/JLLeitschuh/playframework/pull/1/files).

## Usage

### Add to an existing Workflow

Simply add this action to your workflow **after** having checked out your source tree and **before** running any Gradle build:

```yaml
uses: gradle/wrapper-validation-action@v2
```

### Add a new dedicated Workflow

Here's a sample complete workflow you can add to your repositories:

**`.github/workflows/gradle-wrapper-validation.yml`**
```yaml
name: "Validate Gradle Wrapper"
on: [push, pull_request]

jobs:
  validation:
    name: "Validation"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: gradle/wrapper-validation-action@v2
```

## Contributing to an external GitHub Repository

Since [GitHub Actions](https://github.com/features/actions)
are completely free for open source projects and are automatically enabled on almost all projects,
adding this check to a project's build is as simple as contributing a PR.
Enabling the check requires no overhead on behalf of the project maintainer beyond merging the action.

You can add this action to your favorite Gradle based project without checking out their source locally via the
GitHub Web UI thanks to the 'Create new file' button.

![GitHub 'Create new file' Button bar picture](https://user-images.githubusercontent.com/1323708/73676469-6c023c00-4682-11ea-8c0a-5a1e2d29b17f.png)

Simply add a new file named `.github/workflows/gradle-wrapper-validation.yml` with the contents mentioned above.

We recommend the message commit contents of:
 - Title: `Official Gradle Wrapper Validation Action`
 - Body (at minimum): `See: https://github.com/gradle/wrapper-validation-action`

From there, you can easily follow the rest of the prompts to create a Pull Request against the project.

## Reporting Failures

If this GitHub action fails because a `gradle-wrapper.jar` doesn't match one of our published SHA-256 checksums,
we highly recommend that you reach out to us at [security@gradle.com](mailto:security@gradle.com).

**Note:** `gradle-wrapper.jar` generated by Gradle 3.3 to 4.0 are not verifiable because those files were dynamically generated by Gradle in a non-reproducible way. It's not possible to verify the `gradle-wrapper.jar` for those versions are legitimate using a hash comparison. You should try to determine if the `gradle-wrapper.jar` was generated by one of these versions before running the build.

If the Gradle version in `gradle-wrapper.properties` is out of this range, you may need to regenerate the `gradle-wrapper.jar` by running `./gradlew wrapper`. If you need to use a version of Gradle between 3.3 and 4.0, you can use a newer version of Gradle to generate the `gradle-wrapper.jar`.

If you're curious and want to explore what the differences are between the `gradle-wrapper.jar` in your possession
and one of our valid release, you can compare them using this online utility: [diffoscope](https://try.diffoscope.org/).
Regardless of what you find, we still kindly request that you reach out to us and let us know.

## Resources

To learn more about verifying the Gradle Wrapper JAR locally, see our
[guide on the topic](https://docs.gradle.org/current/userguide/gradle_wrapper.html#wrapper_checksum_verification).
