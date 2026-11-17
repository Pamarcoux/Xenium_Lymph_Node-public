# Xenium Lymph Node

Analysis of a 10x Xenium lymph node section (reactive FFPE): from the segmented
transcript table to clustered cells and spatial neighbourhoods.

Segmentation is done upstream with [sopa](https://github.com/prism-oncology/sopa),
whose Snakemake workflow runs proseg with the 10X Genomics segmentation as prior.
The configuration used is kept here as `config/sopa_proseg.yaml`; it is the
upstream `workflow/config/xenium/proseg.yaml` (MIT, see `LICENSE`).

This is a notebook project, not a package. There is no importable module and no
CLI: open the notebook and run it. The notebooks 05 to 15 used for the published
analysis are not included, only the processing template.

## Workflow

```
Xenium bundle (10X)
        |
        |  sopa workflow, config/sopa_proseg.yaml
        v
proseg-output.zarr          SpatialData: transcripts, cell_boundaries, table
        |
        |  notebooks/00_processing.ipynb
        v
cells x genes AnnData        layers counts + lognorm, PCA, UMAP, Leiden
        |
        v
annotation and spatial analysis (not included)
```

## Install

```bash
uv sync
```

`pyproject.toml` pins the versions that matter for spatialdata, squidpy and novae:
`zarr<3`, `xarray<=2024.7`, `dask<2025`. They are in `[tool.uv] override-dependencies`
and must not be bumped without re-running the notebook.

Python 3.11 is required.

## Data

The repository contains no data. Place the sopa output where
`config/xenium.yaml` points to:

```
data/10X/Lymph_Node/<bundle>_outs/proseg-output-v2.zarr
```

The zarr must contain a `table` with raw counts in `X` (uint32, one row per cell,
one column per gene), `obsm["spatial"]`, and the shapes and points layers written
by sopa. If you start from a raw Xenium bundle, run the sopa workflow first:

```bash
git clone https://github.com/prism-oncology/sopa.git
cd sopa/workflow
snakemake --configfile=/path/to/this/repo/config/sopa_proseg.yaml \
          --config data_path=/path/to/xenium_bundle \
          --cores 8 --use-conda
```

## Configuration

`config/xenium.yaml`:

| Key | Meaning |
|-----|---------|
| `input.proseg_zarr` | SpatialData zarr written by sopa |
| `output.h5ad` | output AnnData |
| `qc.min_counts` | minimum transcripts per cell |
| `qc.max_cells` | random subsample size, `null` to keep all cells |
| `clustering.n_top_genes` | highly variable genes used for PCA |
| `clustering.n_pcs`, `n_neighbors` | PCA and neighbour graph size |
| `clustering.resolutions` | Leiden resolutions to compute |
| `clustering.key_prefix` | prefix of the created `obs` columns |
| `spatial.neighbour_radius_px` | radius used later for spatial neighbours |

`qc.max_cells` exists to keep a quick run on a laptop. Set it to `null` for the
full section, which is about 700 000 cells and needs roughly 20 GB of RAM with the
clustering steps.

## Output

`notebooks/00_processing.ipynb` writes an AnnData with:

| Slot | Content |
|------|---------|
| `X` | log-normalised expression |
| `layers["counts"]` | raw transcript counts |
| `layers["lognorm"]` | copy of the log-normalised matrix |
| `obs` | sopa cell metadata plus one `leiden_<res>` column per resolution |
| `obsm` | `spatial`, `X_pca`, `X_umap` |
| `var` | `highly_variable` |

Cell types are not assigned automatically. Labels have to come from a marker
heatmap or an external annotation before any phenotype analysis.

## Not included

The notebooks 05 to 15 (niche definition, differential expression, ligand-receptor
analysis, figures), their builder scripts, the gene set files and all data are
kept private.

## Citations

Segmentation:

> Blampey Q., Mulder K., Gardet M., et al. Sopa: a technology-invariant pipeline
> for analyses of image-based spatial omics. *Nature Communications* 15, 4981
> (2024). doi:10.1038/s41467-024-48981-z

Study this analysis belongs to:

> Marcoux P., Gava F., Tosolini M., et al. Patient-derived lymphoma spheroids
> reveal predictive markers of glofitamab resistance in relapsed/refractory
> B-NHL. *Blood* 147(23), 2770 (2026). doi:10.1182/blood.2025031309

## License

MIT, see `LICENSE`. It covers the code only. No data is distributed.
`config/sopa_proseg.yaml` is copied from the sopa project and keeps its MIT
license and attribution.

## Contact

Paul Marcoux, Inserm / CRCT, Toulouse, `paul.marcoux@inserm.fr`
