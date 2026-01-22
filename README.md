# OSDR-ST (Xenium OSDR notebook)

This repo contains a working notebook to run OSDR on a Xenium dataset and score proliferation/apoptosis markers.

## What’s in here
- `xenium_OSDR.ipynb`: main notebook
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

## Notebook workflow (xenium_OSDR.ipynb)
1. **Load your `.h5ad`**
   - Update the path in the first data-load cell.
2. **Score proliferation / apoptosis**
   - Uses marker sets (Ki67/Top2a/MCMs for proliferation; Caspases/Bcl2 family for apoptosis).
   - Scores are computed on `ad.X` (already normalized/log-transformed).
3. **Build `single_cell_df` for OSDR**
   - Required columns: `x`, `y`, `cell_type`, `division`, `img_id`, `subject_id`.
   - `cell_type` is set from `ad.obs['anno_L2']`.
   - `division` is set to the top 2% by `prolif_score` (adjustable).
   - `subject_id` defaults to `ad.obs['subject_id']` or falls back to `ad.obs['sample_id']`.
   - `img_id` defaults to `ad.obs['fov_id']` or falls back to `ad.obs['sample_id']`.
   - Coordinates are assumed to be in **microns** and converted to meters via `tdm.utils.microns`.
4. **Run OSDR**
   - Uses `Analysis(..., polynomial_dataset_kwargs={'degree': 1}, model_kwargs={'regularization_alpha': 1e-3}, enforce_max_density=False)`
   - These settings avoid singular fits and max-density overflows on large datasets.
5. **Plot phase portraits**
   - Set `cell_types_to_model` (two types) or enable `auto_pick` to plot all pairs.

## Common issues
- **IORegistryError when reading .h5ad**: update `anndata`/`h5py` to the versions above.
- **“Expected a single subject ID”**: ensure `img_id` groups do **not** mix subjects.
- **“Categorical has no attribute item”**: cast `img_id`/`subject_id` to string (already handled in notebook).
- **Overflow in maximal density enforcer**: disable with `enforce_max_density=False`.
- **Singular matrix**: use `degree=1` and add `regularization_alpha`.

## Next steps
- Tune the division cutoff (e.g., top 1%–5%)
- Use a real FOV/section column for `img_id` if available
- Cache analysis results for faster iteration

If you want additional plots or a more automated pipeline, extend `xenium_OSDR.ipynb` accordingly.
