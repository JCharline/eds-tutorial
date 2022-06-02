# eds-tutorial

## About

In this tutorial we introduce some issues related to the analysis of real world data that is made available for research in clinical data warehouses (CDW). It is targeted towards data scientists that master the basics of Python programming and data analysis. The tutorial is decomposed in a series of small exercises and a final project. Whereas small exercises illustrate specific issues, the final project mimics an end-to-end research study that may be reported in a scientific article.

Data is fake, and this project can consequently be freely shared without impacting patients’ privacy. A fake data generator is made available and can be tuned to illustrate various use cases. Its development has been freely inspired by the characteristics and issues observed while analyzing data of the Greater Paris University Hospitals’ CDW, the Entrepôt des Données de Santé (EDS).

## Getting started

### Environment and kernel creation

Python, JupyterLab and an environment manager are recommended. You may choose for instance [Anaconda](https://docs.anaconda.com/anaconda/install/index.html).

First clone the project locally :
`git clone {URL}`

If you use Conda as an environment manager, create a new Python environment with the required packages:
1. `conda create -n eds-tuto python=3.7`
2. `conda activate eds-tuto`
3. `pip install -r requirements.txt`

Create and name a Jupyter kernel related to this virtual environment:
4. `pip install --user ipykernel`
5. `python -m ipykernel install --user --name eds_tutorial`
A kernel named eds_tuto is now available in your jupyterlab!

NB: For VS Code users, in order to see clearly the plots, it is recommended to enable the Theme Matplotlib Plots in your setting > Extensions > Jupyter.

### Scientific libraries installation

The following scientific library developed in the context of Paris’ CDW may moreover be leveraged to facilitate the resolution of some exercises:
- *edsnlp*: rule-based natural language processing pipelines adapted to French clinical notes

To install these libraries:
1. `conda activate eds-tuto`
2. `pip install edsnlp`

The documentation of the library `edsnlp` can be found [here](https://pypi.org/project/edsnlp/)
