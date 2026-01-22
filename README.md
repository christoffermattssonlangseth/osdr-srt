# OSDR-ST (Xenium OSDR notebooks)

This repo contains working notebooks to run OSDR on a Xenium dataset and score proliferation/apoptosis markers.

## What’s in here
- `xenium_OSDR_happy_path.ipynb`: minimal, end‑to‑end workflow (recommended start)
- `xenium_OSDR.ipynb`: full workflow with extra diagnostics and plots
- `osdr_one_shot.ipynb`: copy of the official OSDR tutorial (optional)
- `osdr_quickstart.ipynb`: minimal import test (optional)

## Environment
We used a Python 3.11 conda kernel with:
- `anndata==0.12.7`
- `h5py==3.15.1`
- `scanpy==1.11.5`

Create the kernel:
```bash
conda create -n osdr-py311 python=3.11 pip ipykernel
conda activate osdr-py311
pip install anndata==0.12.7 h5py==3.15.1 scanpy==1.11.5
pip install git+https://github.com/JonathanSomer/osdr.git
python -m ipykernel install --user --name osdr-py311 --display-name "Python (osdr-3.11)"
```

## Happy path (xenium_OSDR_happy_path.ipynb)
1. Load your `.h5ad` data
2. Score proliferation markers (using normalized/log data in `ad.X`)
3. Build `single_cell_df` in OSDR format
4. Choose two cell types
5. Run OSDR and plot phase portrait + growth rate

Key defaults:
- `cell_type` from `ad.obs['anno_L2']`
- `division` = top 2% by `prolif_score`
- `img_id` = `fov_id` if present, else `sample_id`
- `subject_id` = `subject_id` if present, else `sample_id`
- coordinates assumed in **microns** (converted to meters)

## Full workflow (xenium_OSDR.ipynb)
Includes:
- calibration plots
- cross‑validation (toggle)
- growth rate + trajectory plots
- model diagnostics
- pairwise phase portraits for multiple cell types

## Common issues
- **IORegistryError when reading .h5ad**: update `anndata`/`h5py` to the versions above.
- **“Expected a single subject ID”**: ensure `img_id` groups do **not** mix subjects.
- **“Categorical has no attribute item”**: cast `img_id`/`subject_id` to string (handled in notebooks).
- **Overflow in maximal density enforcer**: disable with `enforce_max_density=False`.
- **Singular matrix**: use `degree=1` and add `regularization_alpha`.
- **Unknown cell type names in plots**: add fallbacks to `CELL_TYPE_TO_FULL_NAME` and `CELL_FULL_NAME_TO_COLOR`.

## Next steps
- Tune the division cutoff (e.g., top 1%–5%)
- Use a real FOV/section column for `img_id` if available
- Cache analysis results for faster iteration

