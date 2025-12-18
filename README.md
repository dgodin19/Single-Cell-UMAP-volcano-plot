### Structure of this repository

Below is the report for this project. All code for this project is in the associated jupyter-notebook in the top level directory. 

### Introduction

Ovarian cancer has been difficult to treat with standard immunotherapy approaches. Treatments like PD-1 inhibitors, CAR-T cells, and cancer vaccines have shown some promise in other cancers, but they generally produce weak or short-lived responses in ovarian cancer. One reason is that ovarian tumors tend to have an immunosuppressive environment and relatively few T cells that naturally recognize the tumor.
Chiello et al. looks at a different strategy based on Bispecific T-cell Engagers (BiTEs). BiTEs are engineered molecules that link a T cell to a tumor cell, which can force the T cell to kill the tumor even if it wouldn’t normally recognize it. In this study, the authors take this a step further by engineering T cells to secrete a BiTE that targets folate receptor-alpha (FRα), a surface protein that is commonly expressed in ovarian cancer. These engineered cells, called FR-B T cells, are designed to directly kill FRα⁺ tumor cells and also activate other T cells in the tumor, and potentially broaden the overall immune response.
To understand how these engineered T cells affect the tumor microenvironment, the authors analyze ovarian tumors using single-cell RNA sequencing. This allows them to track changes in different immune cell populations after treatment. For this project, three figures from the paper were reproduced using the processed single-cell dataset they provided. 

### Methods

For this project, I downloaded the processed Seurat object associated with GEO accession GSE280954. The RDS file was loaded in R 4.4.3 using Seurat 5.3.1 and SeuratObject 5.0.2. I extracted the cell-type metadata using dplyr 1.1.4, exported it to a CSV file, and used this metadata later when working in Python. The remaining analyses were performed in Python 3.13.9 with Scanpy 1.11.1. In Python, I downloaded the GSE280954_RAW.tar archive from GEO and unpacked it into separate folders corresponding to each experimental condition.
The Jupyter notebook for this project is structured in a straightforward way. At the top, I include the article citation and the set of modules required for the analysis. Below that is a configuration section containing the parameters reported in the authors’ methods section titled,  “scRNA-seq raw data processing, quality control for cell inclusion in murine studies”. According to the paper, the authors filtered for genes expressed in more than three cells and retained cells that expressed more than 300 genes. They also removed cells with over 80,000 total RNA counts or with more than 10% mitochondrial gene content. Doublets were detected using Scrublet with a threshold of 0.2. For dimensionality reduction, they selected 41 principal components which were determined by an elbow plot, and generated UMAP embeddings using 29 clusters at a Leiden resolution of 0.08. I adopted these same settings for my analysis in order to match the figures as closely as possible.
Following the configuration section, the notebook loads the raw count matrices and concatenates them into a single AnnData object. I wrote a small helper function to standardize cell barcodes so that the AnnData index matched the metadata extracted from Seurat. I included a short R snippet in the notebook showing how the cell-type metadata was originally obtained, followed by the Python code used to merge these annotations into the AnnData object.
The rest of the notebook consists of modular functions for preprocessing (quality-control filtering, normalization, and HVG selection), dimensionality reduction (PCA and UMAP), and figure generation. I reproduced Figure 3B (UMAP), Figure 6A (volcano plot), and Figure 6F (bar plots comparing selected cell populations across treatment conditions). The UMAP and differential expression analysis for the volcano plot were both done in scanpy. 
The main package versions used were: scanpy 1.11.1, scrublet 0.2.3, pandas 2.3.3, numpy 2.3.5, matplotlib 3.10.8, scikit-learn 1.5.2, scikit-image 0.25.2, scikit-misc 0.5.2, and python 3.13.9.

### Results

For Figure 3B, my UMAP embedding did not fully match the authors’ version. In my analysis, I identified five major clusters, whereas the paper reported approximately seven. My clusters grouped together as follows: (1) CD4 T cells, naïve T cells, CD8 T cells, and NK cells; (2) proliferating lymphoid cells, activated B cells, plasma cells, ambiguous B cells, and naïve B cells; (3) proliferating tumor cells, cancer-associated fibroblasts (CAFs), progressive tumor cells, and primary tumor cells; (4) mast cells; and (5) proliferating myeloid cells, unassigned cells, macrophages, basophils, erythrocytes, and neutrophils. In contrast, the authors’ UMAP showed the following structure: one T/NK cell cluster containing Treg, CD4 effector-memory, naïve T, exhausted T, CD8 activated, CD8 effector, and NK cells; a separate B cell cluster containing naïve B cells, activated B cells, and plasma cells; a tumor cluster containing progressive, proliferating, and primary tumor cells; a cluster containing mast cells, basophils, and CAFs; a macrophage-specific cluster; and two distinct neutrophil clusters.
For Figure 6A, I reproduced the volcano plot comparing primary versus progressive tumor cells, using primary tumors as the downregulated group and progressive tumors as the upregulated group. Only one overlapping upregulated gene between my results and the authors’ was Krt7, and for the primary tumors the only matching downregulated gene was Iftim1. The remaining differentially expressed genes did not overlap, likely due to differences in analysis methods.

For Figure 6F, I was able to closely match the authors’ reported cell-type proportions across conditions. My bar plots showed more activated B cells in the DR (durable response) group, more CD8 effector cells in the d28 + Vax group, and more CD4 effector-memory cells in the DR group. I also observed an increased number of Arg1⁺ macrophages in the PD (progressive disease) group and elevated CXCL13-producing macrophages in the DR group. Progressive tumor cells were most abundant in the PD group. In these plots, DR corresponds to the durable response condition, PD to early progressive disease, d28+Vax to the FRBplusPD1-Vax group, and d28−Vax to the FRBplusPD1-last group. The FRBplusPD1 treatment represents combined FR-B T cell therapy with anti–PD-1, and the ±Vax designation indicates whether the booster vaccine was included.
Several factors likely contributed to the discrepancies between my reproduced Figure 3B and 6A and those presented in the paper. The authors performed their analysis in R, whereas my workflow used Scanpy in Python. More specifically, the paper used the MAST model for differential expression, while I used sc.tl.rank_genes_groups with a Wilcoxon test, and different DEG algorithms often produce different gene lists. For the UMAP, even though I used the cell-type labels extracted directly from the Seurat object, the authors appear to have made additional annotation adjustments. Furthermore, UMAP has a stochastic component, so some variation between embeddings is expected.
Looking up the genes in GeneCards for the DEGs in Figure 6A, Krt7 is known to be expressed during the differentiation of simple and stratified epithelial tissues, which aligns with its presence in progressive ovarian tumor cells. Ifitm1, on the other hand, belongs to a family of interferon-induced antiviral proteins and may be a general biomarker with higher expression in primary ovarian cancer cells. These patterns are consistent with the biological context, as it is plausible for Krt7 to be upregulated in cells associated with ovarian cancer tumor progression, while Ifitm1 is more prominent in primary ovarian cancer tumor cells.

### Figures

[3b](3B.png)

UMAP of all cells for all conditions.

[6a](6A.png)

Volcano plot comparing differentially expressed genes in primary tumors and progressive tumors.

[6f](6F.png)

Bar plots of cell proportions for B.Actv, Cd8.Eff, Cd4.EM, TAM.Arg1, TAM.Cxcl13, and Tu-
mor.Prog compared to total cells within the condition. Conditions were DR, d28+VAX, d28-Vax,
and PD.

### Conclusion 

Using the publicly available dataset from GEO, I was able to largely reproduce the main findings of the paper, including UMAP embeddings, differential gene expression, and cell-type proportion analyses. This project allowed me to apply techniques learned in BF550 and BF528 to work with single-cell RNA-seq data in Python using Scanpy, while also developing modular, well-structured code for preprocessing, analysis, and figure generation.	

### References

Chiello JL, Shaikh N, Jacobi J, Gaulin N, Santos G, Keck C, Hess SM, Lichty B, Singh PK, Rosario
SR, Abrams SI, Zsiros E, Long MD, McGray AJR. BiTE-secreting T cells rationally combine with
PD-1 blockade and vaccine boosting to reshape antitumor immunity in ovarian cancer. Mol Ther.
2025 Sep 30:S1525-0016(25)00818-4. doi: 10.1016/j.ymthe.2025.09.047. Epub ahead of print.
PMID: 41029902.
