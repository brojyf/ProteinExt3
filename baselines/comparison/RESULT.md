# Result
Using proteins from the UniProtKB/Swiss-Prot dataset released on or after January 1, 2025, after removing proteins that appear in the CAFA6 training set. There're 1828 proteins in the test set.

## MF

| Method | Fmax | Smin | AUPR | AUC |
| --- | ---: | ---: | ---: | ---: |
| SPROF-GO | **0.493565** | **21.846062** | **0.432958** | **0.844374** |
| DeepGO-SE | 0.454275 | 23.535453 | 0.375135 | 0.822907 |
| ProteinExt3 | 0.327288 | 24.992546 | 0.180707 | 0.626949 |
| ProteinExt3-ProtT5 | - | - | - | - |
| ProteinExt3-ESM2 | - | - | - | - |
| ProteinExt3-CNN | - | - | - | - |

## CC

| Method | Fmax | Smin | AUPR | AUC |
| --- | ---: | ---: | ---: | ---: |
| SPROF-GO | 0.442377 | 11.948879 | **0.349563** | **0.892985** |
| DeepGO-SE | 0.406782 | 12.169886 | 0.326759 | 0.885854 |
| ProteinExt3 | **0.474759** | **10.048571** | 0.331671 | 0.788425 |
| ProteinExt3-ProtT5 | - | - | - | - |
| ProteinExt3-ESM2 | - | - | - | - |
| ProteinExt3-CNN | - | - | - | - |

## BP

| Method | Fmax | Smin | AUPR | AUC |
| --- | ---: | ---: | ---: | ---: |
| SPROF-GO | **0.322181** | **41.693220** | **0.185598** | 0.712674 |
| DeepGO-SE | 0.287973 | 43.354286 | 0.164493 | **0.836181** |
| ProteinExt3 | 0.260408 | 44.898192 | 0.092449 | 0.612776 |
| ProteinExt3-ProtT5 | - | - | - | - |
| ProteinExt3-ESM2 | - | - | - | - |
| ProteinExt3-CNN | - | - | - | - |


## Thresholds

| Category | Method | Fmax Threshold | Smin Threshold |
| --- | --- | ---: | ---: |
| MF | SPROF-GO | 0.208000 | 0.333000 |
| MF | DeepGO-SE | 0.291000 | 0.460000 |
| MF | ProteinExt3 | 0.301000 | 0.301000 |
| MF | ProteinExt3-ProtT5 | - | - |
| MF | ProteinExt3-ESM2 | - | - |
| MF | ProteinExt3-CNN | - | - |
| BP | SPROF-GO | 0.335000 | 0.387000 |
| BP | DeepGO-SE | 0.500000 | 0.632000 |
| BP | ProteinExt3 | 0.300000 | 0.382000 |
| BP | ProteinExt3-ProtT5 | - | - |
| BP | ProteinExt3-ESM2 | - | - |
| BP | ProteinExt3-CNN | - | - |
| CC | SPROF-GO | 0.513000 | 0.871000 |
| CC | DeepGO-SE | 0.749000 | 0.946000 |
| CC | ProteinExt3 | 0.368000 | 0.526000 |
| CC | ProteinExt3-ProtT5 | - | - |
| CC | ProteinExt3-ESM2 | - | - |
| CC | ProteinExt3-CNN | - | - |
