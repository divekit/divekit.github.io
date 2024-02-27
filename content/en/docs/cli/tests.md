---
title: "Tests"
linkTitle: "Tests"
weight: 2
description: >
    Before the CLI can be used in production, it is necessary to test it. This page describes how to test the CLI.
---

### What should be tested in this project?

Given that this CLI is the entry point for the user to interact with Divekit, it is essential to test all commands.
Currently, there is only one command `patch`, but all commands should be tested with the following aspects in mind:

* ***Command Syntax***: Verify that the command syntax is correct
* ***Command Execution***: Ensure that executing the command produces the expected behavior or output
* ***Options and Arguments***: Test each option and argument individually to ensure they are processed correctly
  and test various combinations of options and arguments
* ***Error Handling***: Test how the command handles incorrect syntax, invalid options, or missing arguments

Additionally, testing the utility functions is necessary, as they are used throughout the entire project.
For that the following aspects should be considered:
* ***Code Paths***: Every possible path through the code should be tested, which should include "happy paths"
  (expected input and output) as well as "edge cases" (unexpected inputs and conditions).
* ***Error Conditions***: Check that the code handles error conditions correctly. For example, if a function is
  supposed to handle an array of items, what happens when it’s given an empty array? What about an array with only one
  item, or an array with the maximum number of items?

### How should something be tested?

Commands should be tested with integration tests since they interact with the entire project.
Integration tests are utilized to verify that all components of this project work together as expected
in order to test the mentioned aspects.

To detect early bugs, utility functions should be tested with unit tests. Unit tests are used to
verify the behavior of specific functionalities in isolation. They ensure that individual units of code produce the
correct and expected output for various inputs.

### How are tests written in Go?

#### Prerequisites

It's worth mentioning that the following packages are utilized in this project for testing code.

##### The testing package

The standard library provides the testing package, which is required to support testing in Go. It offers different
types from the testing library [[1, pp. 37-38]](#references):

* `testing.T`: To interact with the test runner, all tests must use this type. It contains a method for
  declaring failing tests, skipping tests, and running tests in parallel.

* `testing.B`: Similar to the test runner, this type is a benchmark runner. It shares the same methods for failing
  tests, skipping tests and running benchmarks concurrently. Benchmarks are generally used to determine performance
  of written code.

* `testing.F`: This type generates a randomized seed for the testing target and collaborates with the `testing.T` type
  to provide test-running functionality. Fuzz tests are unique tests that generate random inputs to discover
  edge cases and identify bugs in written code.

* `testing.M`: This type allows for additional setup or teardown before or after tests are executed.

##### The testify toolkit

The testify toolkit provides several packages to work with assertions, mock objects and
testing suites <a href="https://www.jetbrains.com/help/go/using-the-testify-toolkit.html" target="_blank">[4]</a>.
Primarily, the assertion package is used in this project for writing assertions more easily.

#### Test signature

To write unit or integration tests in Go, it is necessary to construct test functions following
a particular signature:

```go
func TestName(t *testing.T) {
// implementation
}
```

According to this test signature highlights following requirements [[1, p.40]](#references):

* Exported functions with names starting with "Test" are considered tests.
* Test names can have an additional suffix that specifies what the test is covering. The suffix must
  also begin with a capital letter. In this case, "Name" is the specified suffix.
* Tests are required to accept a single parameter of the `*testing.T` type.
* Tests should not include a return type.

#### Unit tests

Unit tests are small, fast tests that verify the behavior of specific functionalities in isolation.
They ensure that individual units of code produce the correct and expected output for various inputs.

To illustrate unit tests, a new file named `divide.go` is generated with the following code:

```go
package main

func Divide(a, b int) float64 {
	return float64(a) / float64(b)
}
```

By convention tests are located in the same package as the function being tested.
It's important that all test files must end with `_test.go` suffix to get detected by the test runner.

Accordingly `divide_test.go` is also created within the main package:

```go
package main

import (
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestDivide(t *testing.T) {
	// Arrange
	should, a, b := 2.5, 5, 2
	// Act
	is := divide(a, b)
	// Assert
	assert.Equal(t, should, is, "Got %v, want %v", is, should)
}
```

Writing unit or integration tests in the Arrange-Act-Assert (AAA) pattern is a common practice. This pattern establishes
a standard
for writing and reading tests, reducing the cognitive load for both new and existing team members and enhancing
the maintainability of the code base [[1, p. 14]](#references).

In this instance, the test is formulated as follows:

* ***Arrange***: All preconditions and inputs get set up.

* ***Act***: The ***Act*** step executes the actions outlined in the test scenario, with the specific actions depending
  on the type of test. In this instance, it calls the Add function and utilizes the inputs from the ***Arrange*** step.

* ***Assert***: During this step, the precondition from the ***Arrange*** step is compared with the output.
  If the output does not match the precondition, the test is considered failed, and an error message is displayed.

It's worth noting that the ***Act*** and ***Assert*** steps can be iterated as many times as needed, proving beneficial,
particularly in the context of table-driven tests.

#### Table-driven tests for unit and integration tests

To cover all test cases it is required to call ***Act*** and ***Assert*** multiple times. It would be possible to write
one test per case, but this would lead to a lot of duplication, reducing the readability. An alternative approach is to
invoke the same test function several times. However, in case of a test failure, pinpointing the exact point of
failure may pose a challenge
<a href="https://blog.jetbrains.com/go/2022/11/22/comprehensive-guide-to-testing-in-go/#writing-a-simple-unit-test"
target="_blank">[2]</a>.
Instead, in the table-driven approach, preconditions and inputs are structured as a table in the ***Arrange*** step.

As a consequence `divide_test.go` gets adjusted in the following steps [[1, pp. 104-109]](#references):

##### Step 1 - Create a structure for test cases

In the first step a custom type is declared within the test function. As an alternative the structure could be declared
outside the scope of the test function. The purpose of this structure is to hold the inputs and expected preconditions
of the test case.

The test cases for the previously mentioned `Divide` function could look like this:

```go
package main

import (
	"math"
	"testing"
)

func TestDivide(t *testing.T) {
	// Arrange
	testCases := []struct {
		name     string  // test case name
		dividend int     // input
		divisor  int     // input
		quotient float64 // expected
	}{
		{"Regular division", 5, 2, 2.5},
		{"Divide with negative numbers", 5, -2, -2.5},
		{"Divide by 0", 5, 0, math.Inf(1)},
	}
}

```

The `struct` type wraps `name`, `dividend`, `divisor` and `quotient`. `name` describes the purpose of a test case
and can be used to identify a test case, in case an error occurs.

##### Step 2 - Executing each test and assert it

Each test case from the table will be executed as a subtest. To achieve this, the `testCases` are iterated over and
each `testCase` is executed in a separate goroutine
<a href="https://golangdocs.com/goroutines-in-golang" target="_blank">[3]</a> with `t.Run()`.
The purpose of this is to individually fail tests without concerns about disrupting other tests.

Within `t.Run()`, the ***Act*** and ***Assert*** steps get performed:

```go
package main

import (
	"github.com/stretchr/testify/assert"
	"math"
	"testing"
)

func TestDivide(t *testing.T) {
	// Arrange
	testCases := []struct {
		name     string  // test case name
		dividend int     // input
		divisor  int     // input
		quotient float64 // expected
	}{
		{"Regular division", 5, 2, 2.5},
		{"Divide with negative numbers", 5, -2, -2.5},
		{"Divide by 0", 5, 0, math.Inf(1)},
	}

	for _, testCase := range testCases {
		t.Run(testCase.name, func(t *testing.T) {
			// Act
			quotient := Divide(testCase.dividend, testCase.divisor)
			// Assert
			assert.Equal(t, testCase.quotient, quotient)
		})
	}
}

```

#### Setup and teardown

##### Setup and teardown before and after a test

Setup and teardown are used to prepare the environment for tests and clean up after tests have been executed.
In Go the type `testing.M` from the testing package fulfills this purpose and is used as a parameter for the
`TestMain` function, which controls the setup and teardown of tests.

To use this function, it must be included within the package alongside the tests, as the scope for functions
is limited to the package in which it is defined. This implies that each package can only have one
`TestMain` function; consequently, it is called only when a test is executed within the package
<a href="https://medium.com/goingogo/why-use-testmain-for-testing-in-go-dafb52b406bc " target="_blank">[5]</a>.

The following example illustrates how it works [[1, p. 51]](#references):

```go
package main

func TestMain(m *testing.M) {
	// setup statements
	setup()

	// run the tests
	e := m.Run()

	// cleanup statements
	teardown()

	// report the exit code
	os.Exit(e)
}

func setup() {
	log.Println("Setting up.")
}
func teardown() {
	log.Println("Tearing down.")
}
```

`TestMain` runs before any tests are executed and defines the `setup` and `teardown` functions. The `Run` method
from `testing.M` is used to invoke the tests and returns an exit code that is used to report the success or failure
of the tests.

##### Setup and teardown before and after ***each*** test

In order to teardown after each test, the `t.Cleanup` function can be used provided by the testing package
<a href="https://blog.jetbrains.com/go/2022/11/22/comprehensive-guide-to-testing-in-go/#test-teardown-and-cleanup"
target="_blank">[2]</a>.
Since there is no mention to `setup` before each test, it can be assumed that the `setup` function is
called at the start of a test.

This example shows how this can be used:

```go
package main

func TestWithSetupAndCleanup(t *testing.T) {
	setup()

	t.Cleanup(func() {
		// cleanup logic
	})

	// more test code here
}

```

#### Write integration tests

Integration tests are used to verify the interaction between different components of a system. However, the mentioned
principles for writing unit tests also apply to integration tests. The only difference is that integration tests involve
a greater amount of code, as they encompass multiple components.

### How to run tests?

To run tests from the CLI, the `go test` command is used, which is part of the Go toolchain
<a href="https://www.thegowiki.com/wiki/Go_Toolchain" target="_blank">[6]</a>.
The list shows some examples of how to run tests:

* To run a specific test, the `-run` flag can be used. For example, to run the `TestDivide` test from
  the `divide_test.go` file, the following command can be used: `go test -run TestDivide`. Note that the argument for
  `-run` is a regular expression, so it is possible to run multiple tests at once.

* To run all tests in a package, run `go test <packageName>`. Note that the package name should include a relative path
  if
  the package is not in the working directory.

* To run all tests in a project, run `go test ./...` . The argument for test is a wildcard, matching all
  subdirectories; therefore, it is crucial for the working directory to be set to the root of the project
  to recursively run all tests.

Additionally, tests can be run from the IDE. For example, in GoLand, the IDE will automatically detect tests
and provide a gutter icon to run them
<a href="https://www.jetbrains.com/help/go/performing-tests.html" target="_blank">[7]</a>.

### How the command `patch` is tested?

#### Prerequisites

Before `patch` can be tested, it is necessary to do the following:

1. Replace the placeholders in the file `.env.example` and rename it to `.env`. If you have no api token, you can
   generate one <a href="https://git.archi-lab.io/profile/personal_access_tokens" target="_blank">here</a>.
2. Run the script `setup.ps1` as administrator. This script will install all necessary dependencies
   and initialize the ARS-, Repo-Editor- and Test-Origin-Repository.

#### Test data

To test `patch`, it was necessary to use a
<a href="https://git.archi-lab.io/staff/testing/divekit-origins/generic-origin/divekit-origin-test-repo" target="_blank">
test origin repository</a> as test data. In this context the test origin repository is a repository that contains
all the necessary files and configurations from ST1 to test different scenarios.

Additionally, a <a href="https://git.archi-lab.io/staff/thomas-testgroup" target="_blank">test group</a> was created
to test if the Repo-Editor-repository actually pushes the generated files to remote repositories.
Currently, the test group contains the following repositories:

```
coderepos:
    ST1_Test_group_8063661e-3603-4b84-b780-aa5ff1c3fe7d
    ST1_Test_group_86bd537d-9995-4c92-a6f4-bec97eeb7c67
    ST1_Test_group_8754b8cb-5bc6-4593-9cb8-7c84df266f59

testrepos:
    ST1_Test_tests_group_446e3369-ed35-473e-b825-9cc0aecd6ba3
    ST1_Test_tests_group_9672285a-67b0-4f2e-830c-72925ba8c76e
```

#### Structure of a test case

`patch` is tested with a table-driven test, which is located in the file `patch_test.go`.

The following example shows the structure of a test case:

```go
package patch

func TestPatch(t *testing.T) {
	testCases := []struct {
		name           string
		arguments      PatchArguments  // input
		generatedFiles []GeneratedFile // expected
		error          error           // expected
	}{
		{
			"example test case",
			PatchArguments{
				dryRun:       true | false,
				logLevel:     "[empty] | info | debug | warning | error",
				originRepo:   "path_to_test_origin_repo",
				home:         "[empty] | path_to_repositories",
				distribution: "[empty] | code | test",
				patchFiles:   []string{"patch_file_name"},
			},
			[]GeneratedFile{
				{
					RepoName:    "repository_name",
					RelFilePath: "path_to_the_generated_file",
					Distribution: Code | Test,
					Include:     []string{"should_be_found_in_the_generated_file"},
					Exclude:     []string{"should_not_be_found_in_the_generated_file"},
				},
			},
			error: nil | errorType,
		},
	}

	// [run test cases]
}

```

The `name` field is the name of the test case and is used to identify the test case in case of an error.

The struct `PatchArguments` contains all the necessary arguments to run the `patch` command:

* `dryRun`: If true, generated files will not be pushed to a remote repository.
* `logLevel`: The log level of the command.
* `originRepo`: The path to the test origin repository.
* `home`: The path to the divekit repositories.
* `distribution`: The distribution to patch.
* `patchFiles`: The patch files to apply.

The struct `GeneratedFile` is the expected result of the `patch` command and contains the following properties:

* `RepoName`: The name of the generated repository.
* `RelFilePath`: The relative file path of the generated file.
* `Distribution`: The distribution of the generated file.
* `Include`: Keywords that should be found in the generated file.
* `Exclude`: Keywords that should not be found in the generated file.

The `error` field is the expected error of the `patch` command. It can be `nil` when no error is expected or
contain a specific error type if an error is expected.

#### Process of a test case

The following code snippet shows how test cases are processed:

```go
package patch

func TestPatch(t *testing.T) {
	// [define test cases]

	for _, testCase := range testCases {
		t.Run(testCase.name, func(t *testing.T) {
			generatedFiles := testCase.generatedFiles
			dryRunFlag := testCase.arguments.dryRun
			distributionFlag := testCase.arguments.distribution

			deleteFilesFromRepositories(t, generatedFiles, dryRunFlag) // step 1
			_, err := executePatch(testCase.arguments)                 // step 2

			checkErrorType(t, testCase.error, err) // step 3
			if err == nil {
				matchGeneratedFiles(t, generatedFiles, distributionFlag) // step 4
				checkFileContent(t, generatedFiles)                      // step 5
				checkPushedFiles(t, generatedFiles, dryRunFlag)          // step 6
			}
		})
	}
}

```

Each test case runs the following sequence of steps:

1. `deleteFilesFromRepositories` deletes the specified files from their respective repositories. Prior to testing,
   it is necessary to delete these files to ensure that they are actually pushed to the repositories,
   given that they are initially included in the repositories.

2. `executePatch` executes the patch command with the given arguments and return the output and the error.

3. `checkErrorType` checks if the expected error type matches with the actual error type.

4. `matchGeneratedFiles` checks if the found file paths match with the expected files and throws an error when 
   there are any differences.

5. `checkFileContent` checks if the content of the files is correct.

6. `checkPushedFiles` checks if the generated files have been pushed correctly to the corresponding repositories.

### References

[1]
A. Simion,
<a href="https://www.amazon.de/Test-Driven-Development-practical-idiomatic-real-world/dp/1803247878" target="_blank">
Test-Driven Development in Go</a>
Packt Publishing Ltd,
2023

[2]
<a href="https://blog.jetbrains.com/go/2022/11/22/comprehensive-guide-to-testing-in-go" target="_blank">
"Comprehensive Guide to Testing in Go | The GoLand Blog,"</a>
The JetBrains Blog
(accessed Jan. 29, 2024).

[3]
<a href="https://golangdocs.com/goroutines-in-golang" target="_blank">"Goroutines in Golang - Golang Docs,"</a>
(accessed Jan. 29, 2024).

[4]
<a href="https://www.jetbrains.com/help/go/using-the-testify-toolkit.html" target="_blank">"Using the Testify toolkit |
GoLand,"</a>
GoLand Help.
(accessed Jan. 29, 2024).

[5]
<a href="https://medium.com/goingogo/why-use-testmain-for-testing-in-go-dafb52b406bc" target="_blank">"Why use TestMain
for testing in Go?"</a>
(accessed Jan. 29, 2024).

[6]
<a href="https://www.thegowiki.com/wiki/Go_Toolchain" target="_blank">"Go Toolchain - Go Wiki"</a>
(accessed Jan. 29, 2024).

[7]
<a href="https://www.jetbrains.com/help/go/performing-tests.html" target="_blank">"Run tests | GoLand,"</a>
GoLand Help.
(accessed Jan. 29, 2024).