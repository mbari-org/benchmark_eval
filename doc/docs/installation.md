# Installation

## Cloning the Repository

In your terminal, type in the command below. Make sure you're in the directory you'd like this to be in!
```
git clone https://github.com/mbari-org/benchmark_eval.git
```

Then, to get into the repository, type
```
cd benchmark_eval
```

## Setting Up The Environment

First, install Python 3.10.13

```
pyenv install 3.10.13
```

If you don't have virtualenv installed, please type 
```
pip install virtualenv
```

Then, create a new virtual environment to work in
```
pyenv virtualenv 3.10.13 insert-env-name
```

Activate your virtual environment using
```
pyenv activate insert-env-name
```

Lastly, install the benchmark_eval_requirements.txt file
```
pip install -r benchmark_eval_requirements.txt
```
*You'll see that there is also a requirements.txt file in the repository. That was from the original TrackEval, but it's a bit outdated and doesn't have updated installations. 


