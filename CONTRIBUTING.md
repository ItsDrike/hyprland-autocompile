# Contributing

## Installation

This project uses [`poetry`](https://python-poetry.org/docs/), which is a tool for managing dependencies for projects
in python virtual environments. You will need to install it to set up this project for development. (On arch linux, you
can do `pacman -S python-poetry`, otherwise follow the liked documentation.)

Once installed, install the project's dependencies (and create a new virtual environment for it) with:

```bash
poetry install
```

After that, it's recommended to activate the environment before working with it, you should do this every time you wish
to do further development on the project. Some IDEs are capable of doing this for you automatically when you open the
project, if yours isn't, you can run:

```bash
poetry shell
```

You can then start your IDE from the terminal, after you ran this command, and it should pick up the python environment
created by poetry.

## Style guide

This project uses pretty strict code formatting rules which are enforced by various linters or auto formatters. To
contribute to the project, you will need to familiarize yourself with these linters.

This project currently uses the following tools for static analysis and formatting:
[`ruff`](https://beta.ruff.rs/docs/), [`black`](https://black.readthedocs.io/en/stable/),
[`isort`](https://pycqa.github.io/isort/), [`pyright`](https://microsoft.github.io/pyright/#/).

### Ruff

Ruff is an alternative program to the more popular [`flake8`](https://flake8.pycqa.org/en/latest/) linter. Ruff is
faster (written in rust! 🦀) and includes most of the popular flake8 extensions directly.

You can check the rules that we're using in [`pyproject.toml`](./pyproject.toml) file, under the `[tools.ruff]`. We
enable most of the rules available, so it is be pretty strict as far as linters go.

To run `ruff` on the code, you can use `ruff check .`, while in the project's root directory (from an activated poetry
environment, alternatively `poetry run ruff .`). Ruff also supports some automatic fixes to many violations it founds,
to enable fixing, you can use `ruff check --fix`.

If you find a rule violation in your code somewhere, and you don't understand what that rule's purpose is, `ruff` evens
supports running `ruff rule [rule id]` (for example `ruff rule ANN401`). These explanations are in markdown, so I'd
recommend using a markdown renderer, ideally `glow` (`pacman -S glow`) and piping the output into it for a much nicer
reading experience: `ruff rule ANN401 | glow`.

### Isort

Isort is a tool for sorting python imports. We do this as it's much easier to orient in files with a lot of imports,
when they are grouped into categories (builtin, external, internal), and the categories are then alphabetically sorted.

Isort can format your imports automatically, so you don't need to put in a lot of manual work when using it, you can
just run `isort .` (or `poetry run isort .`).

### Black

Black is the code autoformatter of our choice, with pretty sensible (though opinionated) rules. All of black's
formatting is compatible with `ruff` linter, so no conflicts between these tools will arise. In fact, black can often
fix a lot of ruff violations on it's own.

To run black, simply execute `black .` (or `poetry run black .`)

### Pyright

Pyright is a type checker (and a language server). If you've never worked with typed codebases before, you might find
this the most difficult to get used to, but in general, it can statically check the code for any typing mistakes. For
example, in python, we can optionally specify type hints (though in our codebase, we actually require that they're
specified), see this example:

```python
def add_numbers(x: int, y: int) -> int:
    return x + y
```

Since python is a dynamic language, these type hints would generally just be decorational, it's nice to have, since
they inform the developer what type should be passed into the function, but the code won't fail if something else is
passed in.

With pyright, it can look at our code, understand that this function expects 2 integers, and gives back an integer, and
if somewhere in the code we would have `add_numbers("a", "b")`, it would get marked as an issue. This can tremendously
help with quickly uncovering bugs, and pyright can do that before you even run the code! We use a pretty strict
configuration of pyright, so you will very likely face some issues during development, but it shouldn't be too hard to
overcome them.

To run pyright, you can use `pyright .` (or `poetry run pyright .`)

### Editor integration

Obviously, it would be pretty annoying to have to run all of these tools manually every time you make a change. But
thankfully, most IDEs do support all of these tools, and can do all of this for you manually. It's still important to
know how to run these when you need to, but generally your editor should be able to do all of the work for you.

If you're using neovim, I would recommend setting up LSP (Language Server Protocol), and installing Pyright, as it has
language server support built into it. Same thing goes with `ruff`. As for `isort` and `black`, you'll want to use
something like `null-ls` for those.

On vscode, you should install the `pylance` plugin, and enable type checking mode in settings (just set
`"python.analysis.typeCheckingMode": "basic"` in `settings.json``). As for the other tools, you should be able to find
extensions for each of those, however as I'm not a vscode user, I can't really help you here (feel free to update this
guide with the exact steps though!)

### Pre-Commit

Other than editor integration, you can (and should) use a tool called `pre-commit`, which we have configuration for in
the project already (see [`.pre-commit-config.yaml`](./.pre-commit-config.yaml)). This tool is capable of creating a
[git hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks), which will run pre-commit each time you try to
create a new commit with git.

That means that pre-commit will run all of the configured linters for the projects automatically before letting you
actually create a new commit, and if issues are detected, the commit will be aborted, and you will see exactly what
linter failed, and with what message. This allows you to never accidentally make commits that break our code style
standards.

To install pre-commit as git hook, you can run `pre-commit install` (or `poetry run pre-commit install`). You only need
to do this once, after setting up the project, and pre-commit will run before each commit automatically. (Only do this
after `poetry install`, as the pre-commit tool itself is a dependency listed in the pyproject.toml manifest)

Sometimes, you may need to actually commit a change that beaks our linting rules. This should be very rare! But
sometimes it is indeed necessary. However if you attempt to do this, you will notice that pre-commit will not actually
let you. To bypass pre-commit, you can pass the `--no-verify` argument when making the commit (e.g.: `git commit -m "My
message" --no-verify`), which will skip all pre commit hooks, and let you create the commit immediately, without
running any checks.

## Make great commits

A well-structured git log is key to a project's maintainability; it provides insight into when and why things were done
for future maintainers of the project.

Commits should be as narrow in scope as possible, and perform only one change at a time (so called "atomic" commits).
Commit that does 15 different things at once (some bugfixes, 2 new features and XYZ) will be much harder for
maintainers/reviewers to follow than 15 commits, each doing only the 1 thing, which it describes in the message.

Good commit history has a lot of benefits, such as allowing us to easily use `git blame` and actually find the exact
cause of a problem, rather than just the commit that added a ton of changes, from which it's still unclear what exactly
might be wrong.

You should also avoid making a lot of minor commits for fixing test failures or linting errors. Just lint before you
push, ideally with [pre-commit](#pre-commit). These minor commits clutter the git history a lot, when they could
instead just have been included in the original commit, by changing history before you push.

We've compiled a few resources on making good commits that you can read through for a better understanding on why this
is so important, and how to get it right:

- <https://itsdrike.com/posts/great-commits/>
- <https://chris.beams.io/posts/git-commit/>
- <https://dhwthompson.com/2019/my-favourite-git-commit>
- <https://thoughtbot.com/blog/5-useful-tips-for-a-better-commit-message>
- <https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html>

## Work in Progress PRs

Whenever you add a pull request that isn't yet ready to be reviewed and merged, you can mark it as a draft. This
provides both visual and functional indicator that the PR isn't yet ready to be reviewed and merged.

This feature should be utilized instead of the traditional method of prepending `[WIP]` to the PR title.

Methods of marking PR as a draft:

1. When creating it

   ![image](https://user-images.githubusercontent.com/20902250/94499351-bc736e80-01fc-11eb-8e99-a7863dd1428a.png)

2. After it was created

   ![image](https://user-images.githubusercontent.com/20902250/94499276-8930df80-01fc-11eb-9292-7f0c6101b995.png)

For more info, check the GitHub's documentation about this feature
[here](https://github.blog/2019-02-14-introducing-draft-pull-requests/)
