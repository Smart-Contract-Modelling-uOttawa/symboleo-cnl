# Symboleo CNL

This repository contains artifacts and processes for generating a CNL for Symboleo refinements.

The CNL is based on analyzing a dataset of real contracts by identifying existing norm refinements and the Symboleo operations to which they correspond. 

Our dataset of interest is [CUAD](https://www.atticusprojectai.org/cuad), which contains a contracts from the [EDGAR database](https://www.sec.gov/edgar/search-and-access). 

This repo includes a set of Python notebooks that walk through the CNL generation process, which involves a certain degree of manual analysis. We provide a high-level overview of the notebooks and artifacts here, with the notebooks themselves containing more detailed information.

The notebooks should be run in [Google Colab](https://colab.research.google.com/) which provides a Python environment pre-loaded with lots of useful functionality, meaning you do not need to download anything locally. However, a Google account is required.

## Assumptions

This phase of analysis assumes that we have already completed two earlier research phases. The first phase involves identifying the operations in Symboleo that we are interested in handling. This set of operations is restricted to operations that _refine_ an existing Symboleo norm. Examples of such operations include temporally refining 
predicates and adding conditional statements. The result of this initial phase is a list of viable Symboleo operations.

The second phase involves general linguistic analysis, where we consider the semantics of the operations identified in RQ1. For each operation, we use linguistic theory to identify NL patterns that may correspond to each of these operations. For example, if we want to identify all of the linguistic structures that correspond to the Symboleo oepration of refining a Happens predicate into a HappensBefore predicate, we look at different ways of expressing the concept of one event occurring _before_ another event in natural language. The result of this is a working CNL, which can be thought of as a set of heuristics for identifying relevant NL refinements that correspond to Symboleo operations.

These operations from RQ1 and the heuristics from RQ2 are pre-requisites for this phase of analysis.

## Walkthrough

### Step 1: Filtering CUAD

The first step of the process involves basic filtering on the CUAD dataset. The initial dataset contains 510 contracts of varying lengths. Since this is novel work, and for the purpose of facilitating manual analysis, we filter the set of contracts to those that contain between 500-3000 words. This is performed by the first notebook (1_cuad_filter).

### Step 2: Extracting Norms

The filtered set of contracts is now analyzed for potential norms. This is done using a set of trigger words that suggest the presence of norms (should, shall, must, may, obligated, required, etc.) This set of trigger words is a heuristic, which can result in both false negatives and false positives. 

Each potential norm is also checked for keywords that may suggest a refinement. For example, we search each norm for the keyword 'before' (and other synonyms of 'before'), which may be suggestive of a Happens -> HappensBefore refinement in Symboleo. This tagging is useful for making the manual analysis easier.

These steps are performed by the second notebook (2_identify_norms). The result of the second notebook will be a set of potential norms from the set of contracts, which must then be manually analyzed to filter out false positives. 

### Step 3: Norm and Refinement Identification

This manual analysis is the most tedious step of the process. It involves analyzing each potential norm to ensure that it is an _actual_ norm _and_ that the norm contains a _refinement_. For each refined norm, we must then estimate what the resulting Symboleo operation will be. This is where our heuristics from RQ2 can be useful.

Once we have identified the valid norms as well as their corresponding Symboleo operations, we also randomly split this off into a construction set and a test set. The construction set will be used to build our CNL, and the test set will be used to evaluate it. We used an 80-20 split. 

### Step 4: Refinement Pattern Extraction

For the construction set, we now want to extract the general patterns expressed by each of our refinements. This involves identifying primitive Symboleo concepts, such as dates, events, and timespans, and replacing these concepts with generic variables. For example the refinement "within 2 weeks of the contract terminating" would correspond to the general pattern: `within TIMESPAN of EVENT`.

This results of this manual analysis must be stored in a csv-like file for the next step of processing. The columns of the csv should contain the following data:
- contract ID
- initial NL norm
- refinement
- Original Symboleo 
- Updated Symboleo
- Additional_notes
- Refinement pattern

The results of the current iteration are stored in the `manual_analysis` file in the repository. It contains a separate sheet for both the construction set and test set. Note that the sheet is _not_ used as the input for the next notebook. The manual work is done on the sheet, but this sheet must then be downloaded as a csv and saved to the drive in order to be used by the next notebook. 


### Step 5: Pattern Aggregation

This step involves aggregating the results of our manual analysis. The refinements are grouped by the operation and the refinement pattern, and the number of times each grouping appears in our set is counted. For example, if we encounter the `before DATE` refinement pattern corresponding to a `SHappensBefore` operation 5 times in our dataset, then we will have the result: `Happens -> SHappensBefore, before DATE, 5`. This is performed by our third notebook (3_aggregate_patterns). This notebook will be run separately for the construction set and the test set.

### Step 6: Pattern class Identification

The result of the notebook will be another csv that must be manually analyzed. This phase of manual analysis involves identifying larger pattern classes found in the refinement patterns. For example, if we find cases of `before DATE` and `prior to DATE`, and these are found to function synonymously in our dataset, we can group them into the `P_BEFORE_S DATE` pattern class, where `P_BEFORE_S` is a variable that can take the value of "before" or "prior to". These larger pattern classes must be manually specified. The results of this manual analysis should be saved to a csv for input into the final step. The columns should correspond to the following data:
- operation
- refinement pattern
- count of refinement pattern
- manually-identified pattern class
- additional notes

The first 3 columns are generated from the notebook. The pattern class in the fourth column is manually entered. A sample row would look like this:
`Happens -> conditional,	in the event EVENT,	22,	CONDITIONAL_A EVENT`. The results of the current iteration can be found in the `4_input_pattern_classes.csv` file. Once again, the manual work is done in a sheets file, which must be exported and stored as a csv for input into the final notebook

### Step 7: Final CNL aggregation

The final automated step is to aggregate the pattern classes manually identified in the previous step and output the aggregated results. This is done by our final notebook (4_final_patterns). The result of this is a csv file containing each mapping candidate and the number of times it appears in the dataset. For example, a result of `P_DURING TIME_PERIOD: Happens -> HappensWithin,	7` means that the pattern class `P_DURING TIME_PERIOD` resulted in the `HappensWithin` refinement 7 times in our construction set. This provides evidence for constructing the final CNL. 

## Evaluation

Steps 4 through 7 can be repeated with the test set to perform an evaluation of the pattern coverage of the constructed CNL. For a given refined norm, we want to check if it is covered by an existing pattern class, and if it maps to the Symboleo operation predicted by the construction set. Any test examples that fail these checks would suggest potential additions to the CNL.



