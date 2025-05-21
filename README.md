# FetMRQC SR

FetMRQC SR is the super-resolution extension of FetMRQC [[paper1](https://arxiv.org/pdf/2304.05879.pdf),[paper2](https://arxiv.org/pdf/2311.04780.pdf)] is a tool for quality assessment (QA) and quality control (QC) of T2-weighted (T2w) fetal brain MR images. 

It builds on top of the utilities developed in the [FetMRQC repository](https://github.com/Medical-Image-Analysis-Laboratory/fetmrqc), a tool for the QC of low-resolution T2w scans.

It contains the tools needed

It consists of two parts.
1. A **rating interface** (visual report) to standardize and facilitate quality annotations of T2w fetal brain MRI images, by creating interactive HTML-based visual reports from fetal brain scans. It uses a pair of low-resolution (LR) T2w images with corresponding brain masks to provide snapshots of the brain in the three orientations of the acquisition in the subject-space. 
2. A **QA/QC model** that can predict the quality of given super-resolution reconstructed volumes. 

Given a list of SRR images listed using `qc_list_bids`, it then uses `srqc_segmentation` to compute the segmentations using BOUNTI [1](#1) and extracts image quality metrics (IQMs) using `srqc_compute_iqms`. These IQMs can then be transformed in FetMRQC SR predictions using `srqc_inference`.

If you have found this useful in your research, please cite 
XXXXX

## Installing FetMRQC_SR
To install FetMRQC SR, just create a new `conda` environment with python 3.9.0

```
conda create --name fetmrqc_sr python=3.9.0
```

Then, simply activate the environment and install `fetmrqc_sr` and its dependencies by running `pip install -e .`

## Custom model training using FetMRQC SR
You can train your custom random forest model to predict from a given list of IQMs and using your own data. This can be done by the following steps.

1. Given a [BIDS-formatted](https://bids.neuroimaging.io/index.html) dataset, get a CSV list of the data with `qc_list_bids` (use `--help` to see the detail). You will need to use the option `--skip_masks`.
2. Once you have your csv file, you can generate the visual reports for manual annotations using  
```
qc_generate_reports --bids_csv <csv_path> --out_dir <output_directory> --sr
```
3. You can then run `qc_generate_index` to generate an index file to easily navigate the reports.
4. Once your ratings are done, you can get back a CSV file using `qc_ratings_to_csv`
5. You can then compute brain segmentations using `srqc_segmentation` and IQMs using `srqc_compute_iqms`. 
6. You will then have everything that you need to train your custom models: manual ratings with automatically extracted IQMs. Using `srqc_train_model`, you will then be able to train your own model.

## Image quality metrics
FetMRQC_SR relies on 106 IQMs, that can be regrouped as follows:


| Intensity-based IQMs              | Count | Description |
|----------------------|-------|-------------|
| _rank_error_ [2]          | 2     | Measure the compressibility of the image using a low-rank approximation |
| _slice-wise difference_ [2,3,4]| 14    | Metrics for outlier rejection: normalized MAE, mutual information, cross-correlation, RMSE, PSNR, SSIM, joint entropy []~\citep{kuklisova-murgasova_2012,kainz_2015,ebner_2020} |
| _entropy_ [5]| 1    | Entropy across the brain volume |
| _sstats_  [5]             | 7     | Mean, median, std, percentiles (5%, 95%), coefficient of variation, kurtosis on brain ROI |
| _filter_image_ [6]        | 2     | Sharpness estimation using Laplace and Sobel filters ([ref fetmrqc]) |
| **Masked-based IQMs**              |  |  |
| _mask_volume_ [6] | 1     | Volume of the brain mask |
| _centroid_ [7]   | 1     | Location of the brain mask centroid |
| **Segmentation-based IQMs**              |  |  |
| _sstats_ [5]                        | 40    | Stats per region (WM, GM, CSF, BS, CBM): mean, median, 5th/95th percentile, kurtosis, std, MAD, voxel count |
| _volume_ [5]                       | 5     | Volume of WM, GM, CSF, BS, CBM |
| _SNR_ [8]                          | 6     | Signal-to-noise ratio per region and globally |
| CNR [9] | 1     | Contrast-to-noise ratio between GM and WM |
| CJV [10]   | 1     | Coefficient of joint variation of GM and WM |
| WM2MAX [5]                     | 1     | Ratio of white matter to maximum intensity |
| **Topology-based IQMs**              |  |  |
| Betti numbers BN0, BN1, BN2 [11]  | 18    | Betti numbers describing the number of connected components (BN0), the number of loops or holes (BN1) and the number of cavities (BN2) for five brain regions (WM, GM, CSF, BS, CBM) and entire brain  |
| Euler Characteristic [11] | 6    | Euler characteristic (= BN0 - BN1 + BN2) for each region and entire brain |

## References

1. Uus, Alena U., et al. "BOUNTI: Brain vOlumetry and aUtomated parcellatioN for 3D feTal MRI." bioRxiv (2023).
2. Kainz, Bernhard, et al. "Fast volume reconstruction from motion corrupted stacks of 2D slices." IEEE transactions on medical imaging 34.9 (2015): 1901-1913.
3. Kuklisova-Murgasova, Maria, et al. "Reconstruction of fetal brain MRI with intensity matching and complete outlier removal." Medical image analysis 16.8 (2012): 1550-1564.
4. Ebner, Michael, et al. "An automated framework for localization, segmentation and super-resolution reconstruction of fetal brain MRI." NeuroImage 206 (2020): 116324.
5. Esteban, Oscar, et al. "MRIQC: Advancing the automatic prediction of image quality in MRI from unseen sites." PloS one 12.9 (2017): e0184661.
6. Sanchez, Thomas, et al. "FetMRQC: A robust quality control system for multi-centric fetal brain MRI." Medical image analysis 97 (2024): 103282.
7. de Dumast, Priscille, et al. "Translating fetal brain magnetic resonance image super-resolution into the clinical environment." European Congress of Magnetic Resonance in Neuropediatrics. 2020.
8. Dietrich, Olaf, et al. "Measurement of signal‐to‐noise ratios in MR images: influence of multichannel coils, parallel imaging, and reconstruction filters." Journal of Magnetic Resonance Imaging: An Official Journal of the International Society for Magnetic Resonance in Medicine 26.2 (2007): 375-385.
9. Magnotta, Vincent A., Lee Friedman, and FIRST BIRN. "Measurement of signal-to-noise and contrast-to-noise in the fBIRN multicenter imaging study." Journal of digital imaging 19 (2006): 140-147.
10. Ganzetti, Marco, Nicole Wenderoth, and Dante Mantini. "Intensity inhomogeneity correction of structural MR images: a data-driven approach to define input algorithm parameters." Frontiers in neuroinformatics 10 (2016): 10.
11. Hu, Xiaoling, et al. "Topology-preserving deep image segmentation." Advances in neural information processing systems 32 (2019).



## License
XXXXX

## Acknowledgements
XXXX

