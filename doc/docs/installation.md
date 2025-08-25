# Installation

## Cloning the Repository

In your terminal, type in the command below. Make sure you're in the directory you'd like this to be in:

```bash
git clone https://github.com/mbari-org/benchmark_eval.git
```

Then, to get into the repository, type:

```bash
cd benchmark_eval
```

## Setting Up The Environment

First, install Python 3.10.13

```bash
pyenv install 3.10.13
```

Then, create a new virtual environment to work in:

```bash
pyenv virtualenv 3.10.13 benchmark_eval
```

!!! note 
    If you don't have `virtualenv` installed, run `pip install virtualenv` or `brew install virtualenv` (macOS) first.

Activate your virtual environment using:

```bash
pyenv activate benchmark_eval
```

Lastly, install the `benchmark_eval_requirements.txt` file:

```bash
pip install -r benchmark_eval_requirements.txt
```

!!! note
    You'll see that there is also a `requirements.txt` file in the repository. That was from the original TrackEval, but it's a bit outdated and doesn't have updated installations.
