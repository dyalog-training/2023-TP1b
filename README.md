# Running Dyalog ]dtest in Docker and GitHub Actions

## Workshop aims

The aims of this workshop is to demonstrate how you can get your tests to run automatically for you 'in the clouds' using [Docker](https://www.docker.com/) and the [GitHub Actions](https://github.com/features/actions) [continous integration](https://en.wikipedia.org/wiki/Continuous_integration) (CI) framework. This means that every time you make a change and push this to the `main` branch of your repository, your tests will run, and the results will become visible in the GitHub web interface without any further actions from you.

If you're not used to using GitHub to host your APL code, worry not -- we'll work it through together. Even if you prefer another source code revisioning system, or another CI framework (like [circle](https://circleci.com/) or [jenkins](https://www.jenkins.io/)), the broad strokes of the approach we'll cover here should hopefully still be relevant to you.

GitHub Actions is free to use for public repositories, and private repositories get a limited, but generous, allocation of free minutes of runner time.

## Pre-requisites

To make the most of this workshop, check that

1. you have [Docker desktop](https://www.docker.com/products/docker-desktop/) installed
2. you have [Dyalog APL v18.2](https://www.dyalog.com/download-zone.htm) installed
3. you have a [GitHub account](https://github.com/join) you can create repositories in
4. you either have [git installed](https://github.com/git-guides/install-git) on your computer, or
5. ..if you're a Windows user, consider installing [GitHub Desktop](https://desktop.github.com/)

If you're on a Mac, install the follwing shell alias to make it easier to run Dyalog from the command line:
```sh
alias dyalog="/Applications/Dyalog-18.2.app/Contents/Resources/Dyalog/mapl"
```

Place this in your shell's expected config file (`~/.bashrc` or `~/.bash_profile` if you use `bash` or `~/.zshrc` if you use `zsh`) and run `source {your-shell-config-file}`.

These are the minimum requirements.

Optionally, there are plenty of auxiliary tools that make working with `git` and `GitHub` more streamlined if you're not a fan of the `git` command line. Some examples are:

1. [GitHub CLI](https://cli.github.com/)
2. [GitHub Desktop](https://docs.github.com/en/desktop/installing-and-authenticating-to-github-desktop/installing-github-desktop). 
3. Mac users may want to consider installing [GitUp](https://gitup.co/)

Many popular IDEs and code editors have excellent `git` and GitHub integration, e.g. [Visual Studio Code](https://code.visualstudio.com/docs/sourcecontrol/overview).

If you do intend to use the GitHub CLI (note: optional), it might be a good idea to get that installed and set up before the day. Same with using `git` over SSH (a good idea, but also optional) -- ensure you have working [SSH keys installed](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) prior to the workshop.

## Get set up

Now let's get this repository into your own account, and then onto your computer.

1. Fork this repository into your own GitHub account, either using
    * the [GitHub web interface](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository), or
    * the [GitHub CLI](https://cli.github.com/): `gh repo fork dyalog-training/2023-TP1b`

2. Clone your fork to your local machine, using one of the following methods, noting the `--recursive` switch to ensure we also clone submodules:
    * Git SSH: `git clone --recursive git@github.com:{YOUR-GITHUB-ACCOUNT}/2023-TP1b.git`
    * Git HTTPS: `git clone --recursive https://github.com/{YOUR-GITHUB-ACCOUNT}/2023-TP1b.git`
    * [GitHub Desktop](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-a-repository-from-github-to-github-desktop)

> ℹ️ **Note**: it's NOT sufficient to clone the workshop repository directly, it must be forked first. The reason for this is that in order to be able to trigger the GitHub Action, you need to be able to push to the remote repository, and if you clone it directly, you won't have sufficient credentials to push.

## Repository contents

This repository contains:

1. A test runner, [run.aplf](https://github.com/dyalog-training/2023-TP1b/blob/main/src/run.aplf)
2. A single function under test (for demo purposes), [mysum.aplf](https://github.com/dyalog-training/2023-TP1b/blob/main/src/mysum.aplf)
3. A directory containing two demo unit tests, written using ]dtest, [tests](https://github.com/dyalog-training/2023-TP1b/tree/main/tests)
4. A [Dockerfile](https://github.com/dyalog-training/2023-TP1b/blob/main/Dockerfile), capable of executing the unit tests
5. A yaml file [test-runner.yml](https://github.com/dyalog-training/2023-TP1b/blob/main/.github/workflows/test-runner.yml), which will use the Dockerfile to run the unit tests as a GitHub Actions workflow.

Any push to the main branch of this repository should trigger the unit tests to run in the workflow.

## Link the code into your workspace

Start Dyalog, and execute the following to link the source code into the session:

    ]link.create # {/path/to}/2023-TP1b/src

Replace `{/path/to}` with the actual path to the `2023-TP1b` you cloned.

## Trigger the tests manually

In the session, use the `]dtest` user command to run the tests. Note that the repository contains the latest version of DTest -- if you didn't install this earlier, copy the file `{/path/to}/2023-TP1b/DBuildTest/DyalogBuild.dyalog` in your `MyUCMDs` folder. 

Run the command:

    ]dtest {/path/to}/2023-TP1b/tests

## Running tests from the command line

The source directory contains a test runner that enables us to trigger the tests from the command line:

```
dyalog -b -s LOAD=src
```
That command exits with a 0 if all went OK, and 11 otherwise.

Try changing the file `src/mysum.aplf` to 
```apl
mysum ← {
    ⍺-⍵
}
```
and show that there is now a test failure. If all tests are still succeeding, this can be a symptom of not using the latest version of `]dtest` -- something in the fork+clone process went awry. Remove the `DBuildTest` directory, and re-clone the sub-module:

```sh
rm -rf DBuildTest
git submodule update --init --recursive
```

## Build the Docker container

```
docker build -t dytest .
```

## Run the unit tests inside the Docker container

Mac/Linux:
```
docker run --rm \
  -v "$(pwd)/DBuildTest/DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" \
  -v "$(pwd)/src:/src" \
  -v "$(pwd)/tests:/tests" \
  dytest
```

In Windows PowerShell:
```
docker run --rm `
  -v "${PWD}/DBuildTest/DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" `
  -v "${PWD}/src:/src" `
  -v "${PWD}/tests:/tests" `
  dytest
```

Using the Windows Command Prompt:
```
docker run --rm -v "%cd%\DBuildTest\DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" -v "%cd%\src:/src" -v "%cd%\tests:/tests" dytest
```

If you see an error of the type:

```
/src does not define the function #.Run
```
then you most likely ran the container from the wrong directory: it must be run from the root of the checked-out repository.

## Git cheat sheet

If you're new to `git`, it can seem daunting. However, it's worth the time investment to learn it, given its ubiquity. Here are some common incantations to use on the command line. For a more in-depth introduction, see GitHub's [git guide](https://github.com/git-guides).

**Clone a repository**

```
git clone [--recursive] [git@github.com:{ACCT}/{REPO}.git | https://github.com/{ACCT}/{REPO}.git]
```

**Stage, and commit modified (and deleted) files**
```
git commit -a -m 'Your well-formed commit message here'
```

**Push branch to remote**
```
git push origin {branch-name}
```

**Make a new local branch, and switch to it**
```
git checkout -b {my-new-branch-name}
```

**Switch to other local branch**
```
git switch {branch-name}
```

**Pull all remote changes in main branch**
```
git pull origin main [--ff-only]
```

## Trouble-shooting: debugging the Docker container

If you experience problems using the Docker container, you can request shell access and examine it manually. Debugging a Docker container can sometimes feel like a dark art. With this container, you can try the following:

In the root of the repository, run the following command to bypass the set entry point to gain shell access (Mac/Linux):
```
docker run --rm --entrypoint /bin/sh \
  -v "$(pwd)/DBuildTest/DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" \
  -v "$(pwd)/src:/src" \
  -v "$(pwd)/tests:/tests" \
  -it dytest
```
In Windows PowerShell:
```
docker run --rm --entrypoint /bin/sh `
  -v "${PWD}/DBuildTest/DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" `
  -v "${PWD}/src:/src" `
  -v "${PWD}/tests:/tests" `
  -it dytest
```
Start with verifying that the mapped files look as they should. Look in the `/src` and `/tests` folders and check that their contents match those of the repository's matching folders. Check that `/home/dyalog/MyUCMDs` contains the file `DBuildTest.dyalog`. Check that the latter is indeed a file containing APL code, and not a directory (a sign that the submodule wasn't correctly cloned).

If that all looks good, let's ensure we can run dyalog manually in interactive mode. For this we need to adjust a couple of environment variables. Execute the following statements:

```sh
unset LOAD
export DYALOG=/opt/mdyalog/18.2/64/unicode/
export LD_LIBRARY_PATH="${DYALOG}:${LD_LIBRARY_PATH}"
export WSPATH=$WSPATH:${DYALOG}/ws
export TERM=xterm-256color
export APL_TEXTINAPLCORE=${APL_TEXTINAPLCORE-1}
export TRACE_ON_ERROR=0
export SESSION_FILE="${SESSION_FILE-$DYALOG/default.dse}"
```

Now see what happens if you try to run the tests from the command-line (inside the container):

```sh
dyalog -b -s LOAD=/src
```
If that does not yield the behaviour you expect, start Dyalog interactively:
```sh
dyalog
```
Link the source code:
```
]link.create # /src
```
and try running `]dtest`:
```
]dtest /tests
```


