+++
title = 'Venv_jupyter'
date = 2025-06-11
draft = false
+++
# Using Jupyer notebook as python application
Not too long ago i accidently discovered that Jupyer is just a python package that you can download and use.\
Having good environment for interanctive programming in few commands on almost any device with python installed is really handy\
It is simple to set up if you work in python on daily basis or used terminal a bit.

## Prerequisite
- python3
- pip3

## How to create virtual python environment
Installing packages randomly on system is kinda dumb idea.\
Let's create python venv (virtual environment)
```bash
python -m venv my_venv
# you can change name to something else
```
## How to activate venv
After python create venv you can activate it in current shell
```bash
# on linux
source my_venv/bin/activate
# or 
. my_venv/bin/activate
```
shell prompt should change\
in my case:
```bash
# before:
talandar@gentoo ~ $
# after:
(my_venv) talandar@gentoo ~ $
```

## Installing jupyter
we can now use python package manager called pip to install jupyter
```bash
pip install jupyter
```
## Running jupyter notebook
to run jupyer notebook just type this:
```bash
jupyter notebook 
```
observe log for link to website\
it will look like this:
```bash
Or copy and paste one of these URLs:
    http://localhost:8888/tree?token=sometoken....
    http://127.0.0.1:8888/tree?token=sometoken....
```
paste in your browser and you are ready to go

## TLDR
```bash
# create and activate venv
python -m venv my_venv
. my_venv/bin/activate
# install jupyter
pip install jupyter
# run jupyter notebook
jupyter notebook 
# observe logs for link to page
Or copy and paste one of these URLs:
    http://localhost:8888/tree?token=sometoken....
```
