Title: pre-commit Advices
Date: 2023-04-26T20:18:17+03:00

## What is pre-commit
At first, let me remind what pre-commit is.
[pre-commit](https://pre-commit.com) is a tool to manage git hooks inside your projects, package them and import them from other sources. Also it make very convinient to manage them using it's declarative YAML configuration.

Git hooks are useful if you have some linters/formatters, and you want to make sure your commited changes are checked by your dev tools.

## Advices

### Prefer system hooks over hooks managed by pre-commit
pre-commit suggests to use their environment management for hooks:
```yaml
-   repo: https://github.com/psf/black
    rev: 22.10.0
    hooks:
    -   id: black
```
Inside `black` repo there is defined hook that will be installed in separate python virtual environment alongside with it's dependencies. It may look good from programmer's perspective, but it introduces two problems:
- You can't run the linter/formatter by yourself. Especially it's useful with formatters, so you have to format code by yourself. Of course you can install the same tool using another package manager, but it leads to another problem.
- If you have tool installed via `pre-commit` and another package manager, then their versions can differ. It applies to list of plugins too, so you can end to have a different set of linters with their plugins inside `pre-commit` and your dev environment.

The solution is simple: **install tools outside pre-commit and use `language: system` to run them**. For example, we're using `poetry` as package manager, so we use something like this:
```yaml
- repo: local
  hooks:
    - id: black
      entry: poetry run black  # poetry run ... ensures to run command inside it's venv
      language: system
```
It uses the same `black` as you have inside `venv`. The same applies to `flake8` and it's plugins.

### pre-commit and CI
CI is a place where our code is checked, built, tested and deployed. And it's important to ensure checks which are ran locally on developers' machines and checks inside CI are the same. So `lint` job should consist of `pre-commit` calling only. 
```bash
pre-commit run --all --verbose
```
Of course base `Docker` image must provide `pre-commit` command. For `Python`-based projects you can use your package manager (`poetry` in dev section for example), install it via `pip` or some other way.
