================================================================================
crispr_protocol.ipynb - README
================================================================================

A single Google Colab notebook that processes plate-reader fluorescence data
for a CRISPR-based diagnostic assay.

The notebook is split into one mandatory Setup section and four largely
independent Scripts (1-4). Once Setup has been executed, each Script can be
run on its own; the only cross-script dependency is the theoretical-LoD cell
at the end of Script 4.


--------------------------------------------------------------------------------
Files in this folder
--------------------------------------------------------------------------------

  crispr_protocol.ipynb        The notebook itself.
  README.txt                   This file - quick overview and hierarchy.
  USER_MANUAL.txt              Section-by-section walkthrough.

  Background.xlsx              Example input for Script 1 (background plate).
  Flatfield.xlsx               Example input for Script 1 (flat-field plate).
  Reporter_calibration.xlsx    Example input for Script 2.1 (calibration).
  Reporter_degradation.xlsx    Example input for Script 2.4 (k_rep).
  VaryingSubstrate.xlsx        Example input for Script 3 (substrate titration).
  VaryingTarget.xlsx           Example input for Script 4 (target titration).


--------------------------------------------------------------------------------
Quick start
--------------------------------------------------------------------------------

  1. Go to https://colab.research.google.com
  2. File -> Upload notebook -> pick crispr_protocol.ipynb
  3. Run Section 0 (Setup) FIRST. It loads every library and helper.
  4. Run any Script (1, 2, 3, 4) - each is independent given Setup.
  5. Each script's first sub-cell prompts you to upload an input file via the
     browser. Click "Choose Files" when the picker appears.

If you want the theoretical LoD (Section 4.6), you must additionally have run
2.2, 2.4.2, 3.3, 3.4.1, 4.1 and 4.2 - see "Cross-script dependencies" below.


--------------------------------------------------------------------------------
Hierarchy
--------------------------------------------------------------------------------

  0. Setup (run me first)
     |   imports + plate helpers (load_384_table, reshape_to_plate, time_scroller)
     |
  1. Script 1 - Flat-field and background correction
     |   1.1 Upload background file              -> BG
     |   1.2 Upload flat-field file              -> FF
     |   1.3 Upload raw data file                -> RAW
     |   1.4 Correct for flat-field & background -> Corrected = (RAW-BG) / (FF-BG)
     |   1.5 Download the corrected data         -> writes .xlsx + browser download
     |
  2. Script 2 - Reporter calibration
     |   2.1 Upload calibration file             -> df (c, cleaved, uncleaved)
     |   2.2 Extract signal-to-conc conversion   -> Fcl_fit, Fucl_fit, c0_fit
     |   2.3 Three operational regimes           -> linear/transition/saturation bounds
     |   2.4 Reporter degradation rate (k_rep)
     |       2.4.1 Upload + plot degradation data    -> df_deg, conc_map_deg
     |       2.4.2 Fit k_rep by linear regression    -> k_slope (= k_rep)
     |
  3. Script 3 - Michaelis-Menten kinetics + self-consistency
     |   3.1 Upload progress curve data (varying substrate) -> df, time_col
     |   3.2 Extract initial reaction rates      -> replicate_rate_df, grouped_rate_df
     |   3.3 Michaelis-Menten analysis           -> mm_results, Vmax, Km
     |   3.4 Self-consistency check
     |       3.4.1 Extract derived vars + compute kcat
     |             -> t_lin_max, max_mean_reaction_velocity,
     |                S0_at_max_velocity, E0 (user input), kcat
     |       3.4.2 alpha, beta, gamma            -> alpha, beta, gamma
     |
  4. Script 4 - Limit of Detection
     |   4.1 Upload progress curve data (varying target) -> df_tar
     |   4.2 Extract endpoint signals            -> target_endpoint_df, time_col
     |   4.3 Determine endpoint LoD Le
     |       4.3.1 NTC threshold P_thr + plot
     |       4.3.2 Interpolate Le at P_thr       -> LoD (experimental Le)
     |   4.4 Extract initial reaction rates      -> replicate_rate_df_tar
     |   4.5 Determine rate-based LoD Lv
     |       4.5.1 NTC threshold Rate_thr + plot
     |       4.5.2 Interpolate Lv at Rate_thr    -> rate_based_LoD (experimental Lv)
     |   4.6 Theoretical LoD                     -> theo_Le, theo_Lv
                                                    (needs Vmax/Km from 3.3,
                                                     kcat from 3.4.1,
                                                     Fcl/Fucl from 2.2,
                                                     k_rep from 2.4.2,
                                                     df_tar from 4.1+4.2)


--------------------------------------------------------------------------------
Cross-script dependencies
--------------------------------------------------------------------------------

Scripts 1-4 are independent of one another EXCEPT for the final cell:

  Section 4.6 (Theoretical LoD) needs:
      Km                from Section 3.3
      kcat              from Section 3.4.1
      Fcl_fit, Fucl_fit from Section 2.2
      k_rep (= k_slope) from Section 2.4.2
      df_tar, time_col  from Sections 4.1 and 4.2

If any of these are missing the cell prints an error naming the missing
variable and stops.

All other sub-sections only depend on earlier sub-sections within the same
Script (e.g. 1.4 needs 1.1-1.3; 3.3 needs 3.2; etc.).


--------------------------------------------------------------------------------
Requirements
--------------------------------------------------------------------------------

Runs on the default Google Colab Python 3 runtime. Required packages (all
preinstalled in Colab): numpy, pandas, matplotlib, scipy, ipywidgets,
IPython.

Local execution is possible if you have Jupyter and those libraries, but the
upload cells (which use google.colab.files.upload) will fail outside Colab.
See USER_MANUAL.txt section 11 for how to adapt them for local use.


--------------------------------------------------------------------------------
Further reading
--------------------------------------------------------------------------------

For per-section purpose, expected file layouts, list of output variables, and
troubleshooting, see USER_MANUAL.txt in the same folder.
