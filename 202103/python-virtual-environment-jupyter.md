# Python虚拟环境与Jupyter联动手册

# Python virtual environment with Jupyter Notebook

## 2021 年 3 月15 日

## Mar 15 2021

---

### create and enter virtual environment

```shell
python3 -m venv <env-name>
source <env-name>/bin/activate
```

### associate with jupyter notebook

```shell
pip install ipykernel
python3 -m ipykernel install --user --name=<env-name>
```

Then you could see the kernel name in right-top of jupyter notebook, and you can change current `.ipynb` file to virtual environment by click 'Kernel -> Change kernel' on the top bar.

```shell
source /path-to-env/<env-name>/bin/activate
cd /path-to-pyfile/
jupyter notebook
```

Note that when you launch jupyter notebook, you must have activated the virtual environment, and it is convenient to operate python files if you are in the folder where your python files stay.

### upgrade all outdated package using pip

```shell
pip list --outdated --format=freeze | grep -v '^\-e' | cut -d = -f 1  | xargs -n1 pip install -U
```

You'd better do this in the virtual environment, since in your root environment it may exist many useless packages that you don't want to upgrade (time-consuming if you do!).



