# Getting started

This guide walks you through downloading these notebooks and running them on your own computer, even
if you have never written a line of code. You do not need to understand Python. You install a few
things once, open the notebook, change a few marked lines, and press a key to run each step.

It works on both Windows and Mac. Where the two differ, both are shown. Read this once from top to
bottom before you start. Setup takes about 30 minutes the first time. After that, opening the
notebook takes under a minute.

If you have any questions, you can reach me (Trina) at xychanxiyue@gmail.com. Thanks!

---

## 1. What you will actually do

There are three notebooks.

- **Notebook 1** turns a raw Stereo-seq chip into a table of cells. This one needs a powerful
  computer with an NVIDIA graphics card, so most people run it on the shared lab machine, or ask
  whoever set it up to run it for them. It is a one time step per sample.
- **Notebook 2** takes the table of cells from Notebook 1 and does the biology: quality control,
  clustering, naming the cell types, finding tissue regions, and making the figures. This one runs
  on a normal laptop and is the one you will run most often.
- **Notebook 3 is optional.** It labels your cells automatically by comparing them to a reference
  dataset, using a tool called cell2location. It also needs a graphics card, so you run it on the
  lab machine or on Google Colab. Use it only if you want automatic cell types instead of naming the
  clusters by hand in Notebook 2. Notebook 3 has its own guide inside it.

If you are new, start with Notebook 2.

---

## 2. Words you will see

- **Terminal.** A window where you type commands instead of clicking buttons. On Mac it is called
  Terminal. On Windows, once you install the software below, you use one called Anaconda Prompt.
- **Conda.** A program that installs Python and science packages for you and keeps them tidy.
- **Environment.** A named box that holds one set of packages. We keep each notebook's packages in
  its own box so they do not clash. Notebook 2 uses a box called `banksy`. You switch into a box
  before you work, with `conda activate`.
- **Jupyter Lab.** The app that opens the notebook in your web browser.
- **Notebook.** The file ending in `.ipynb`. It is a mix of written notes and small blocks of code.
- **Cell.** One block inside a notebook. You run cells one at a time.
- **Kernel.** The engine that runs a notebook. You pick which box the notebook uses by choosing the
  matching kernel. People forget this step, so watch for it.

---

## 3. Get the code onto your computer

You are downloading the code only. The large data files are left out on purpose, so you run the
notebooks on your own sample files.

Pick one of these two ways.

**The simple way, download a ZIP:**

1. Open the repository page in your web browser.
2. Click the green **Code** button, then **Download ZIP**.
3. Find the ZIP in your Downloads and unzip it. On Mac, double click it. On Windows, right click it
   and choose Extract All.
4. You now have a folder with the notebooks and this guide inside. Remember where it is.

**The git way, if you have git installed:**

```bash
git clone https://github.com/misoxy/spatial_pipeline.git
```

That makes a folder called `spatial_pipeline` wherever you ran the command.

---

## 4. One time setup

You do this once per computer.

### 4a. Install Miniconda

1. Go to the Miniconda page: https://docs.conda.io/en/latest/miniconda.html
2. Download the installer for your system:
   - **Windows:** the Windows installer, ending in `.exe`.
   - **Mac:** the macOS installer. Most newer Macs are Apple Silicon (the name has `arm64`). If
     unsure, click the Apple logo at the top left, choose About This Mac, and read the chip line.
3. Open the file you downloaded and follow the prompts. Accept the defaults.

### 4b. Open the right terminal

- **Windows:** click Start, type `Anaconda Prompt`, and open **Anaconda Prompt (miniconda3)**. Use
  this window for every command below, not the plain Command Prompt.
- **Mac:** press Command and Space, type `Terminal`, press Return. Close it and open a fresh one
  after installing, so the change takes effect.

If you see the word `(base)` at the start of the line, conda is working.

### 4c. Make the environment and install Jupyter Lab

In that terminal, paste these lines one at a time, pressing Return after each. The first line builds
the box, the rest fill it. This takes a few minutes. The same commands work on Windows and Mac.

```bash
conda create -n banksy python=3.10 -y
conda activate banksy
pip install scanpy anndata leidenalg scikit-learn python-igraph
pip install jupyterlab
```

If you also need to set up Notebook 1 on a GPU machine, see the setup section in `README.md`. That
one needs an NVIDIA card and is best done by someone technical the first time.

---

## 5. Every time: launch Jupyter Lab

1. Open the terminal (Anaconda Prompt on Windows, Terminal on Mac).
2. Switch into the box:

   ```bash
   conda activate banksy
   ```

3. Move into the folder you downloaded in section 3. The easy way that works on both systems: type
   `cd` and a space, then drag the folder from your file browser (File Explorer or Finder) into the
   terminal window. Its address appears. Press Return.

   ```bash
   cd  (then drag your folder here and press Return)
   ```

4. Start Jupyter Lab:

   ```bash
   jupyter lab
   ```

Your web browser opens with a file list on the left. Leave the terminal open in the background.
Closing it closes Jupyter Lab.

---

## 6. Open the notebook and pick the kernel

1. In the file list on the left, double click `02_downstream_analysis.ipynb`.
2. Look at the top right of the notebook. It shows the current kernel. If it does not say `banksy`,
   click it and choose the `banksy` kernel from the menu. If you skip this, the code will not find
   its packages.

---

## 7. How to run a notebook

A notebook runs from top to bottom, one cell at a time. Do not skip ahead, because later cells
depend on earlier ones.

- Click a cell to select it.
- Press **Shift and Return together** to run that cell and move to the next.
- A running cell shows `[*]` on its left. When it finishes it shows a number.
- Wait for one cell to finish before running the next.

To run the whole notebook, use the top menu: Run, then Run All Cells. While you are still choosing
settings, it is safer to run cell by cell so you can look at each result.

---

## 8. The only lines you change

You do not edit the code. You only change the lines marked with `# EDIT`. Each has a short note next
to it saying what it controls. Leave everything else exactly as it is.

Here is what you must supply for **Notebook 2**, all in the first code cell (Step 1):

| Line | What to put there | Example |
|---|---|---|
| `SAMPLE` | A short name for your sample | `"Y40172EA"` |
| `IN` | The full path to the `.h5ad` file Notebook 1 produced | `"/Users/you/Downloads/Y40172EA_raw.h5ad"` |
| `OUT` | The folder where figures and results are saved | `"/Users/you/Downloads/Y40172EA_out2"` |
| `MIN_COUNTS` | A quality cutoff, set after you look at Step 2 | `150` |
| `MIN_GENES` | A quality cutoff, set after you look at Step 2 | `100` |
| `MAX_PCT_MT` | A quality cutoff, set after you look at Step 2 | `5` |

For **Notebook 1** the inputs are the raw chip files: the `SAMPLE` name, the `.tissue.gef` file (all
the transcripts from the sequencer), and the `_ssDNA_regist.tif` image (the nuclei stain), plus a
folder to write into. Those lines are also in its Step 1, marked `# EDIT`.

### How to get the full path of a file

The path is just the file's address on the computer.

- **Mac:** open Finder, find the file, right click it, hold the Option key, and choose Copy as
  Pathname. Paste it between the quotation marks on the matching `# EDIT` line.
- **Windows:** open File Explorer, hold Shift and right click the file, and choose Copy as path.
  Paste it in. Then two small fixes so Python is happy: make sure there is exactly one pair of
  quotation marks around it, and change every backslash `\` to a forward slash `/`. For example
  `C:\Users\you\file.h5ad` becomes `"C:/Users/you/file.h5ad"`.

---

## 9. Three steps where you look, then decide, then rerun

Most of Notebook 2 just runs. Three steps ask you to look at a picture and set a number. This is
normal.

- **Quality cutoffs (Step 2).** The notebook draws histograms of how many transcripts each cell has.
  Based on the shape, you set `MIN_COUNTS`, `MIN_GENES`, and `MAX_PCT_MT` up in Step 1, then run
  again from Step 4.
- **Number of components (Step 6).** A plot shows where the useful signal drops off. You set `N_PCS`
  to that number.
- **Clustering detail (Step 7).** `RES` controls how many groups the cells split into. Higher gives
  more groups, lower gives fewer.

Two more steps name things, and these must be rewritten for every sample, because the group numbers
come out in a different order each run: `lineage_map` in Step 10 (group number to cell type) and
`domain_map` in Step 16 (region number to region name). The notebook explains how to read the
markers so you know which is which.

---

## 10. What you get out

Notebook 2 writes everything into the `OUT` folder you chose:

- `SAMPLE_annotated.h5ad`, the finished data with cell types and regions saved on it.
- The figures: the cell type map, the dotplot, the UMAP, the region maps, and the tissue specific
  panels.

Change the marked lines, run top to bottom, look at the three checkpoints, and collect your figures.

---

## 11. If something goes wrong

- **A cell shows a red error box.** Read the last line first, it usually names the problem. The most
  common cause is a wrong path. Check the path on the `# EDIT` line is exact, including spelling and
  the `.h5ad` ending. On Windows, check you used forward slashes.
- **`ModuleNotFoundError`.** The notebook is using the wrong box. Go to section 6 and set the kernel
  to `banksy`.
- **`FileNotFoundError`.** The path in Step 1 points at a file that is not there. Copy the path again
  using the trick in section 8.
- **A number or plot looks wrong.** You may have skipped a cell. Use Run, then Run All Cells.
- **Jupyter Lab will not start.** Make sure you ran `conda activate banksy` first, and that you are
  in the right folder with `cd`. On Windows, make sure you are in Anaconda Prompt, not Command
  Prompt.

