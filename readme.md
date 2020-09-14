[![Abcdspec-compliant](https://img.shields.io/badge/ABCD_Spec-v1.1-green.svg)](https://github.com/brain-life/abcd-spec)
[![Run on Brainlife.io](https://img.shields.io/badge/Brainlife-bl.app.346-blue.svg)](https://doi.org/10.25663/brainlife.app.346)
##
## OCT &amp; MAIA Data processing

## Authors

- Daniel Bullock ([dnbulloc@iu.edu](mailto:dnbulloc@iu.edu))
- Jasleen Jolly ([jolly@eye.ox.ac.uk](mailto:jasleen.jolly@eye.ox.ac.uk))

## Lab Directors

- [Franco Pestilli](frakkopesto@gmail.com)
- [Holly Bridge](holly.bridge@ndcn.ox.ac.uk)

(For MAIA readme, please proceed further down the document)

### Funding Acknowledgement
brainlife.io is publicly funded and for the sustainability of the project it is helpful to Acknowledge the use of the platform. We kindly ask that you acknowledge the funding below in your publications and code reusing this code.

### Citations
We kindly ask that you cite the following articles when publishing papers and code using this code. 

1. Avesani, P., McPherson, B., Hayashi, S. et al. The open diffusion data derivatives, brain data upcycling via integrated publishing of derivatives and reproducible open cloud services. Sci Data 6, 69 (2019). [https://doi.org/10.1038/s41597-019-0073-y](https://doi.org/10.1038/s41597-019-0073-y)

#### MIT Copyright (c) 2020 brainlife.io The University of Texas at Austin and Indiana University

## OCT

**Before running this code (OCT)**

Process OCT images from compatible systems with the Orion® software (Voxeleron, LLC, Pleasanton, CA, USA).

Label output csv files with a 3 components in the filename &#39;000\_OD\_T.csv&#39;

The first 3 digits are the patient identifier. Eye is signalled by OD for right and OS for left.

Create a file named &#39;Location.csv&#39; to identify the foveal centre. This must contain the following data with the headings exactly as written:

| Filename | Eye | slice | position | Size X pixel | Size x mm | Position on HEE |
| --- | --- | --- | --- | --- | --- | --- |

Size X pixels and X mm are determined by the OCT scan parameters and may be identical across all images. On the OCT software, such as Heidelberg, identify the foveal centre in the original image. Record the slice this is found in. The final column contains precise X location of the foveal centre. The pixel position (position) is then calculated by (SizeXpixel / SizeXmm) \* Position on HEE.

If you have multiple groups in your dataset set up a group directory. For each group create a
csv file containing a list of the 3 digit identifiers for the participants in that group.

**Running this code (OCT)**

This code is written in Matlab.  It makes use of the _imresize_ function and thus requires Matlab&#39;s image processing toolkit.

The primary wrapper function is _OCTanalysisWrapper __.m_ which goes from individual raw data all the way to group-level analysis.   Thus, once the local github repository has been added to Matlab&#39;s path, running the contents of _OCTanalysisWrapper__.m_ (after setting paths appropriately) should be sufficient to perform the full analysis.
<br/> <br/><br/>
Ensure the correct pathways are set for (see below for further details):

rawSubjDataDir (input CSV files)

targetOutputDir (initial output)

analysisMeanDir (secondary output)

centroidCSVPath (Location file)

groupKeyDir (folder that specifies groups)

groupAnalysisDir (final output)
<br/><br/><br/>
In addition, customise the following to your layers of interest:

layerIndexSequences={1:7,1:4,5,6,5:6};

analysesNames={&#39;TT&#39;,&#39;IR&#39;,&#39;ONL&#39;,&#39;PROS&#39;,&#39;PC&#39;};
<br/><br/><br/>
This is based on the output from Orion:

The data is arranged as all\_layers(width,slices,num\_layers)

1: Reading layer: ILM to RNFL-posterior

2: Reading layer: RNFL-posterior to IPL-posterior

3: Reading layer: IPL-posterior to INL-posterior

4: Reading layer: INL-posterior to OPL-posterior

5: Reading layer: OPL-posterior to ONL-posterior

6: Reading layer: ONL-posterior to RPE

7: Reading layer: RPE to BRUCHS

Therefore, we read 8 layers when no additional layers are added.
<br/><br/><br/>
**Code details (OCT)**

The wrapper,_OCTanalysisWrapper__.m_ , directly calls the following functions:


| **Function name** | **Function description** | **Variable / Parameter inputs** |
| --- | --- | --- |
| parseCsvExport | Provided by Jonathan Oakley, Voxeleron. Allows reading in of segmented thickness measurements and sorting by layer | rawSubjDataDir |
| createCSVsumOutputFiles | Iteratively takes data generated by Voxeleron&#39;s Orion for the accurate segmentation of OCT data, and computes subject-level amalgam csv outputs in accordance with requested layer sequences | rawSubjDataDir,<br/> targetOutputDir,<br/> layerIndexSequences,<br/> analysesNames, |
| analyzeOCTDataWrapper | For each of the layer sequence amalgams generated by createCSVsumOutputFiles, performs an iterative, degree wise analysis and creates a subject-level output csv. | analysisMeanDir,<br/> centroidCSVPath,<br/> visFieldDiam,<br/> theshFloor,<br/> gaussKernel,<br/> meanShape, |
| OCTGroupAnalysisWrapper | Creates group level analysis outputs based on a keyfile identifying group membership. | inputDir,<br/> groupKeyDir,<br/> outputDir |

Several of the variables are simply paths to files or directories containing necessary data/metadata


| **File/Path variable name** | **Contents** | **Structure/Naming convention** |
| --- | --- | --- |
| rawSubjDataDir | Directory containing subject-level data generated by Voxeleron&#39;s [deviceName] | Name(s): &quot;[subjectNum]\_[eyeLabel]\_T.csv&quot; <br/>Structure: TBD |
| targetOutputDir | Directory containing (or to contain) the subject-level amalgam csv outputs generated by createCSVsumOutputFiles | Name(s): &quot;[subjectNum]\_[eyeLabel]\_[layerLabel].csv&quot; , <br/> Structure: TBD |
| analysisMeanDir | Directory containing (or to contain) subject-level analyses (degree iterated means) | Name(s): &quot;[subjectNum]\_[eyeLabel]\_meanTable.csv&quot;&amp;&quot;[subjectNum]\_[eyeLabel]\_stdTable.csv&quot; ,<br/>  Structure: TBD |
| outputDir | Directory containing (or to contain) group-level analyses (degree-wise, cross-subject means) | Name(s): &quot;[eyeLabel]\_GroupMean\_[groupLabel].csv&quot;&amp;&quot;[eyeLabel]\_StdMean\_[groupLabel].csv&quot;<br/> Structure: TBD |
| centroidCSVPath | Path to a specific csv file which contains (among other information) the centroid location of each subject in the entire dataset | Name(s):&quot;Location.csv&quot;Structure: TBD |
| groupKeyDir | A directory counting csv files (named after the group name) containing subject ID&#39;s corresponding to group membership | Name(s):&quot;[groupLabel].xlsx <br/> Structure: TBD |

The rest of the variables are parameters which the user may modify in order to impact their analysis as desired.

| **Parameter variable name** | **Purpose** | **Specifications** |
| --- | --- | --- |
| layerIndexSequences | Indicates which layers are to be summed in createCSVsumOutputFiles to generate the layer amalgams that will be processed later. | Sequence[1] of sequence[2] of integers.  Sequence[1] must be equal to the number of inputs in analysesNames.  sequence[2] must be unique integers which do not exceed the total number of layers obtained from Voxeleron&#39;s [deviceName] |
| analysesNames | Indicates the abbreviation for the layer amalgam, corresponds to [specify standard]. | Set of strings, equal in length to sequence[1] of layerIndexSequences |
| visFieldDiam | Indicates the total _diameter_ of the visual field measured by the experimenter.  Assumed to be consistent across subjects.  The user/experimenter should know this about their data.The code must be amended if this is not the case, e.g. retrospective analysis of clinical data. | Numerical value |
| theshFloor | Specifies the desired threshold to be applied to input data obtained from createCSVsumOutputFiles.   Applied across subjects (prior to computation of means).  Values below this are set to NaN and are not featured in computation of mean.  If no threshold is desired, enter a value of []. | Numerical value |
| gaussKernel | Specifies the desired smoothing kernel to be applied to input data obtained from createCSVsumOutputFiles.   Applied across subjects (prior to computation of means).  Pixel values are thus the result of a gaussian smoothing process.  If no threshold is desired, enter a value of [] or 1. | Odd integer value |
| meanShape | Specifies the desired method for computing the mean.  &quot;rings&quot; will generate averages for concentric, degree-specific rings (like a bullseye), while &quot;full&quot; will compute the mean for the entire visual field (to the specified current, iterated degree). | Either &quot;rings&quot; or &quot;full&quot; |
