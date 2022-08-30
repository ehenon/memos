# Customize git config by folders of projects

## Context

Git configuration settings can be specified with the `git config` command. As explained in the [documentation](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration), Git has 3 configuration levels:
- system (`--system` option)
- user (`--global` option)
- project (`--local` option)

Let's say you have a `dev` directory where all your development projects are located. Maybe you work for several companies, and maybe you like to work on personal projects too? If so, your directory tree might look like this:

```
dev
└───companyA
│   └───project1
│   |   │   ...
│   |   │   ...
|   |
│   └───project2
│       │   ...
│       │   ...
│
└───companyB
│   └───project1
│   |   │   ...
│   |   │   ...
|   |
│   └───project2
│       │   ...
│       │   ...
│
└───perso
│   └───project1
│   |   │   ...
│   |   │   ...
|   |
│   └───project2
│       │   ...
│       │   ...
```

## Problem

In this case, configuring Git at system or user level would not be relevant. You would have to configure Git at the lowest level, i.e. at the project level (using the `--local` option for example). The problem with this solution is that you will have to manage configuration for each Git project manually, which could quickly become very tedious if you have to manage a large number of projects for different companies.

## Solution

An interesting alternative would be to manage Git configuration at the folder level. For example, we would have one configuration defined for the entire personal folder, another for the entire company A folder, another for company B, and so on. Below are the steps to follow to set up this kind of configuration.

Create a `.gitconfig` file at the root of your `perso` folder with this content:

```
[user]
    email = <your_personal_email_address>
```

Do the same with the company directories. Example for the company A:

```
[user]
    email = <your_companyA_email_address>
```

Open the `.gitconfig` file containing your user-level configuration (if you are on Windows, it is certainly in the `C:/Users/<your_name>/` directory) and modify it:

```
[user]
	name = <your_name>
[includeIf "gitdir:<path_of_your_companyA_folder>"]
    path = "<path_of_your_companyA_folder>/.gitconfig"
[includeIf "gitdir:<path_of_your_companyB_folder>"]
    path = "<path_of_your_companyB_folder>/.gitconfig"
[includeIf "gitdir:<path_of_your_perso_folder>"]
    path = "<path_of_your_perso_folder>/.gitconfig"
```

At this point, your directory tree might look like this:

```
dev
└───companyA
|   │   .gitconfig
|   │
│   └───project1
│   |   │   ...
│   |   │   ...
|   |
│   └───project2
│       │   ...
│       │   ...
│
└───companyB
|   │   .gitconfig
|   │
│   └───project1
│   |   │   ...
│   |   │   ...
|   |
│   └───project2
│       │   ...
│       │   ...
│
└───perso
|   │   .gitconfig
|   │
│   └───project1
│   |   │   ...
│   |   │   ...
|   |
│   └───project2
│       │   ...
│       │   ...
│
```

That's all! You have configured Git at the "folder level". No matter how many projects you have or will have in your folders dedicated to company A, company B, or your personal work, your name and email will be automatically configured without having to do it manually.

To verify that the solution worked, you can open a Git Bash, walk through your different projects and check the configured mail like this:

```
<user>@<computer> MINGW64 ~/dev
$ cd companyA/project1

<user>@<computer> MINGW64 ~/dev/companyA/project1
$ git config user.email
<your_companyA_email_address> ✔️

<user>@<computer> MINGW64 ~/dev/companyA/project1
$ cd ../../companyB/project2

<user>@<computer> MINGW64 ~/dev/companyB/project2
$ git config user.email
<your_companyB_email_address> ✔️

<user>@<computer> MINGW64 ~/dev/companyB/project2
$ cd ../../perso/project1

<user>@<computer> MINGW64 ~/dev/perso/project1
$ git config user.email
<your_personal_email_address> ✔️
```
