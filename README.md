# Spatial analysis pipeline

A spatial transcriptomics analysis pipeline. It segments single cells with Cellpose and Proseg, a cleaner
alternative to the default CellBin segmentation, then runs quality control, clustering, and BANKSY tissue
domains.

There are two ways to get cell types: name the clusters from their marker genes (Step 9 in Notebook 2), or map
them from a reference with cell2location (Notebook 3). A keloid skin sample is included as an optional worked
example, but you do not need it.

**Never used Python before? Start with [GETTING_STARTED.md](GETTING_STARTED.md).** It covers installing the
software, environments, launching Jupyter Lab, and what to type in. The rest of this file is the technical
overview.

## The three notebooks

```
tissue.gef  +  ssDNA image
        |
        |   Notebook 1  (GPU machine / WSL, cellpose env)
        |   CPSAM finds nuclei  ->  Proseg builds cells
        v
  SAMPLE_cpsam_proseg_raw.h5ad          one cell per row, gene counts, xy position
        |
        |   Notebook 2  (your laptop, banksy env)
        |   QC -> clustering -> cell types -> BANKSY domains
        v
  SAMPLE_annotated.h5ad  +  figures/

  Optional, for automatic cell types from a reference:
  SAMPLE_cpsam_proseg_raw.h5ad  +  your reference .h5ad
        |
        |   Notebook 3  (GPU machine or Google Colab, cell2loc env)
        |   cell2location maps the reference cell types onto your cells
        v
  SAMPLE_cell2location.h5ad
```

| Notebook | Runs on | Environment | Time |
|---|---|---|---|
| `01_segmentation_cpsam_proseg.ipynb` | A computer with an NVIDIA GPU (WSL Ubuntu) | `cellpose` | minutes of setup, then 10 to 40 min segmentation |
| `02_downstream_analysis.ipynb` | Your own laptop, Windows or Mac | `banksy` | 10 to 20 min, longer with BANKSY |
| `03_cell2location_gpu.ipynb` | A computer with an NVIDIA GPU, or Google Colab | `cell2loc` | tens of minutes on a GPU |

They are split because they need different machines and different software. Notebook 1 needs a computer with
an NVIDIA GPU for the nucleus finder, and the Proseg step inside it is run from the terminal, not from within
the notebook. Notebook 2 only needs scanpy and BANKSY, so it runs fine on a normal personal laptop, Windows or
Mac. Notebook 3 is optional: it also needs a GPU, so run it on the same machine as Notebook 1 or on Google
Colab. Use it only if you want automatic cell types from a reference instead of naming clusters by hand.

## How to run, start to finish

1. **Notebook 1, on the GPU machine.** Open a terminal, `conda activate cellpose`, `jupyter lab`, open the
   notebook, pick the `cellpose` kernel. Edit Step 1 (sample name and the four paths). Run the cells top to
   bottom. It saves `SAMPLE_cpsam_proseg_raw.h5ad`. Step 9 (Proseg) is run in the terminal, the notebook tells
   you the exact command.
2. **Copy the `.h5ad` to your laptop** (into Downloads or wherever you keep samples).
3. **Notebook 2, on your own laptop.** Open a terminal, `conda activate banksy`, `jupyter lab`, open the notebook, pick
   the `banksy` kernel. Edit Step 1 (sample name, the input path, the output folder). Run top to bottom. Name your
   clusters in the generic Step 9. It saves `SAMPLE_annotated.h5ad` and all the figures. Part B is an optional
   keloid example you can read or skip.
4. **Notebook 3 (optional), on a GPU or Colab.** Only if you want automatic cell types from a reference. Edit
   Step 1 (your spatial `.h5ad`, your reference `.h5ad`, and the reference cell type column). Run top to bottom.
   It saves `SAMPLE_cell2location.h5ad` with a cell type per cell.

## The rule for editing

Only change lines marked `# EDIT`. Everything else can stay as it is. Each `# EDIT` line has a short note
saying what the number or path controls.

Three steps ask you to look at a plot, then set a number, then rerun:

- **Quality cutoffs** (Notebook 2, Step 2): look at the histograms, set `MIN_COUNTS`, `MIN_GENES`, `MAX_PCT_MT`
  in the Step 2 set cell, rerun from Step 4.
- **Number of PCs** (Notebook 2, Step 6): look at the scree plot, set `N_PCS` in Step 7.
- **Clustering resolution** (Notebook 2, Step 7): set `RES` higher for more clusters, lower for fewer.

Two things must be set for every sample, because cluster and domain numbers are not stable between runs:

- `cluster_to_type` (Notebook 2, Step 9): cluster number to cell type. This is the generic annotation. Leave it
  empty to accept the automatic best guess, or fill it in to correct the guess.
- `domain_map` (Notebook 2, Step 17): BANKSY domain number to named region.

(The keloid example in Part B has its own `lineage_map` in Step 11. Ignore it unless you are working through that
example.)

## Setup, once per machine

GPU machine (for Notebook 1):

```bash
conda create -n cellpose python=3.10 -y
conda activate cellpose
pip install "cellpose>=3.0" tifffile scikit-image h5py zarr anndata scipy numpy pandas matplotlib
cargo install proseg        # needs the Rust toolchain; the binary lands in ~/.cargo/bin
```

Your own laptop (for Notebook 2), Windows or Mac: make a `banksy` environment with scanpy, anndata, leidenalg,
scikit-learn, and BANKSY. On the lab Mac it already exists at `/opt/miniconda3/envs/banksy`. For the exact
install commands, see [GETTING_STARTED.md](GETTING_STARTED.md).

GPU machine or Google Colab (for Notebook 3, optional): make a `cell2loc` environment with `pip install
cell2location`, which pulls in scvi-tools and PyTorch. Notebook 3's first cells have the exact commands,
including the Colab version.

## What each notebook produces

Notebook 1:
- `cpsam/mask_cpsam_fullroi.npy` the nucleus mask (cached, so reruns are fast)
- `transcripts_fullroi_cpsam_seed.csv.gz` the transcript table Proseg reads
- `proseg_cpsam_voxel_2.0_full/` the Proseg output
- `SAMPLE_cpsam_proseg_raw.h5ad` the input to Notebook 2

Notebook 2 (into the output folder you set in Step 1):
- `SAMPLE_annotated.h5ad` the final object with cell types, regions, and all QC columns
- cell type spatial map, dotplot, marker matrixplot, cell type UMAP
- BANKSY domain maps, region map
- from the optional keloid example: fibroblast subtype map, collagen kept versus blocked control, plasma cell foci map

Notebook 3 (optional):
- `SAMPLE_cell2location.h5ad` your spatial object with a cell type abundance per cell and a `cell2loc_type` label
- a spatial map coloured by `cell2loc_type`

## Notes for a new tissue

The generic path needs almost no changes: run Notebook 2 through Step 9 and edit `broad_markers` and
`cluster_to_type` for your cell types, then BANKSY (Step 16) and `domain_map` (Step 17), then save (Step 22).

For automatic annotation, use Notebook 3 with a reference that matches your tissue (for keloid skin a keloid or
skin reference, for mouse lung a mouse lung reference).

The keloid example in Part B is optional. If you want to adapt it rather than skip it, change these:
- the lineage marker lists in Step 3 (`progs`) and the merger threshold `THR`
- the targeted marker check and `lineage_map` in Steps 10 to 11
- the fibroblast section (Steps 12 to 14) is skin specific; replace with your own compartment of interest or skip
- `markers` and `order` in Step 15
- the niche pairs in Step 18, the foci question in Step 20, the reference atlas in Step 21

## Contact

Maintained by Trina. You can reach me at xychanxiyue@gmail.com if you have further questions.
