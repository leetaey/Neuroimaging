####Running TBSS

Overview: running TBSS involves a few simple steps:

* create FA images from your diffusion study data
* tbss_1_preproc - prepare your FA data in your TBSS working directory in the right format
* tbss_2_reg - apply nonlinear registration of all FA images into standard space
* tbss_3_postreg - create the mean FA image and skeletonise it
* tbss_4_prestats - project all subjects' FA data onto the mean FA skeleton
* stats (e.g., randomise) - feed the 4D projected FA data into GLM modelling and thresholding in order to find voxels which correlate with your model.
We now go through the TBSS steps in detail.  


####create FA data from a diffusion MRI study

#####1. Eddy Current Correction

Eddy currents in the gradient coils induce (approximate) stretches and shears in the diffusion weighted images. These distortions are different for different gradient directions. Eddy Current Correction corrects for these distortions, and for simple head motion, using affine registration to a reference volume.

In the FDT GUI, use the top left drop down menu to select Eddy current correction.

Diffusion weighted data: Use the browse button to select your diffusion weighted dataset (a 4D image).
Corrected output data : Use the browse button to specify a filename for the corrected 4D dataset.
Reference volume : Set the volume number for the reference volume that will be used as a target to register all other volumes to. (default=0, i.e. the first volume)
Command line utility

<pre><code>eddy_correct <4dinput> <4doutput> <reference_no></code></pre>

#####2. Create a brain mask by running bet on one of the B=0 (no diffusion weighting) images

fslroi - extract region of interest (ROI) from an image. You can a) take a 3D ROI from a 3D data set (or if it is 4D, the same ROI is taken from each time point and a new 4D data set is created), b) extract just some time points from a 4D data set, or c) control time and space limits to the ROI. Note that the arguments are minimum index and size (not maximum index). So to extract voxels 10 to 12 inclusive you would specify 10 and 3 (not 10 and 12).

<pre><code>fslroi (original) (target) 0 1</code></pre>

BET (Brain Extraction Tool) deletes non-brain tissue from an image of the whole head. It can also estimate the inner and outer skull surfaces, and outer scalp surface, if you have good quality T1 and T2 input images.

<pre><code>bet (original) (target) -m -n -R -f 0.3 

#####3. Fit the diffusion tensor model using dtifit

DTIFIT fits a diffusion tensor model at each voxel. You would typically run dtifit on data that has been pre-processed and eddy current corrected. Note that dtifit is not necessary in order to use the probabilistic tractography (which depends on the output of BEDPOSTX, not DTIFIT).

In the FDT GUI, use the top left drop down menu to select DTIFIT.

Input: You can specify an input directory containing all the required files with standardized filenames, or alternatively you can specify input files manually by turning on the specify input files manually switch. If an input directory is specified then all files must be named as shown in parentheses below. If input files are specified manually they can have any filename. Required files are:

Diffusion weighted data (data): A 4D series of data volumes. This will include diffusion-weighted volumes and volume(s) with no diffusion weighting.
BET binary brain mask (nodif_brain_mask): A single binarised volume in diffusion space containing ones inside the brain and zeroes outside the brain.
Output basename: User specifies a basename that will be used to name the outputs of dtifit. If the directory input option is used then the basename will be dti.
Gradient directions (bvecs): An ASCII text file containing a list of gradient directions applied during diffusion weighted volumes. The order of entries in this file must match the order of volumes in the input data series.
The format is

<pre><code>x_1 x_2 x_3 ... x_n  
y_1 y_2 y_3 ... y_n  
z_1 z_2 z_3 ... z_n</code></pre>

Vectors are normalised to unit length within the dtifit code. For volumes in which there was no diffusion weighting, the entry should still be present, although the direction of the vector does not matter!

b values (bvals): An ASCII text file containing a list of b values applied during each volume acquisition. The b values are assumed to be in s/mm^2 units. The order of entries in this file must match the order of volumes in the input data and entries in the gradient directions text file.

<pre><code>dtifit -k (dti data file) -m (Bet binary mask file) -o (Output basename)  
-r (b vectors file) -b (b values file) -v (switch on diagnostic messages)</code></pre>

####tbss_1_preproc

You now need to create a new, empty directory (folder) in which you will run the TBSS analysis, for example:


<pre><code>mkdir mytbss</code></pre>
Then copy into there all of your subjects' FA images, giving each subject's FA image a different name. You will make later analysis easier if you name the images in a logical order, for example so that all controls are listed before all patients:


<pre><code>cd mytbss
ls
  CON_N00300_dti_data_FA.nii.gz
  CON_N00302_dti_data_FA.nii.gz
  CON_N00499_dti_data_FA.nii.gz
  PAT_N00373_dti_data_FA.nii.gz
  PAT_N00422_dti_data_FA.nii.gz
  PAT_N03600_dti_data_FA.nii.gz</code></pre>
You are now nearly ready to run the first TBSS script, which will erode your FA images slightly and zero the end slices (to remove likely outliers from the diffusion tensor fitting). Type:


<pre><code>tbss_1_preproc *.nii.gz</code></pre>
(the * expands to the list of input images). The script will move all the processed FA images into a new sub-directory called FA (where all the registration steps will later take place,) and will also create another sub-directory called origdata and place all your original images in there for posterity.

Finally, the script runs `slicesdir`, which creates an overview webpage containing a static view of each of the input images, so that you can then quickly view each of them for obvious problems.


####tbss_2_reg

The next TBSS script (tbss_2_reg) runs the nonlinear registration, aligning all FA images to a 1x1x1mm standard space. The target image used in the registrations can either be a pre-defined target, or can be automatically chosen to be the most "typical" subject in the study. In general we recommend using the FMRIB58_FA standard-space image as the target in TBSS. This involves carrying out just one registration per subject and generally gives good alignment results. This option is applied by using the -T flag. Alternatively, you can supply your own target image by using the -t option.

The third option is to align every FA image to every other one, identify the "most representative" one, and use this as the target image. This target image is then affine-aligned into MNI152 standard space, and every image is transformed into 1x1x1mm MNI152 space by combining the nonlinear transform to the target FA image with the affine transform from that target to MNI152 space. This option is selected by using the -n flag, and is the recommended option if you need to generate a study-specific, for example if the subjects are all young children (and hence the adult-derived FMRIB58_FA target is inappropriate).

Direct registration to the high resolution FMRIB58_FA image takes about 10 minutes x N subjects (when running on a single computer). In comparison, the all-subjects-to-all-subjects option takes about 5 minutes x N x N. Hence the latter approach can take much longer to run than the former.

If your lab has a batch submission system (e.g., SGE) then you may want to setup FSL to take advantage of this - then all the registration jobs get submitted to the batch system for parallel processing. In this case, ask your system administrator to edit $FSLDIR/bin/fsl_sub in order to setup FSL to make use of your cluster environment. Once this is done, running tbss_2_reg submits all the registrations to your batch system, and you need to watch for them all to complete before moving on to the next stage of TBSS.

Once all the registrations have finished running, you are ready to move onto the next step.


####tbss_3_postreg

The next TBSS script applies the nonlinear transforms found in the previous stage to all subjects to bring them into standard space.

If the previous stage was run with the -n option (find the most typical subject as the target), then this script first needs to make the decision about which of your FA images is the most "typical", for selection as the target image to apply all nonlinear transformations into the space of. (The script does this by taking each FA image in turn, and estimating the average amount of warping that was necessary to align all other images to it; it then finds the one that had the smallest amount of average warping when used as a target.) Obviously if you pre-specified the FA target image then this step is automatically skipped. The script then takes the target and affine-aligns it into 1x1x1mm MNI152 space - this resolution is chosen as the later skeletonisation and projection steps work well at this resolution, and the choice of working in MNI152 space is chosen for convenience of display and coordinate reporting later. Once this is done, each subject's FA image has the nonlinear transform to the target and then the affine transform to MNI152 space applied, resulting in a transformation of the original FA image into MNI152 space (actually the two transformations are combined before being applied, to avoid having to resample the image twice).

The above results in a standard-space version of each subject's FA image; next these are all merged into a single 4D image file called all_FA, created in a new subdirectory called stats. Next, the mean of all FA images is created, called mean_FA, and this is then fed into the FA skeletonisation program to create mean_FA_skeleton.

All of the above is done simply by running the script:


<pre><code>tbss_3_postreg -S</code></pre>
Alternatively, if you wish to use the FMRIB58_FA mean FA image and its derived skeleton, instead of the mean of your subjects in the study, use the -T option:


<pre><code>tbss_3_postreg -T</code></pre>
In general we would recommend using the -S option (derive the mean FA and skeleton from the actual subjects you have).

The script finishes by telling you to check whether a suitable threshold for the mean FA skeleton is 0.2 (a typical value used by the next script). For example, load the 4D FA data and the skeleton into FSLView:


<pre><code>cd stats
fslview all_FA -b 0,0.8 mean_FA_skeleton -b 0.2,0.8 -l Green</code></pre>

eg_skel_all_FA.png The -b option sets sensible display range options, and in the case of the skeleton image, also controls the thresholding applied. Now turn on the movie loop; you will see the mean FA skeleton on top of each different subject's aligned FA image. If all the processing so far has worked ok the skeleton should look like the examples shown here (see the TBSS paper for more examples of different subjects' results underneath the skeleton). If the registration has worked well you should see that in general each subject's major tracts are reasonably well aligned to the relevant parts of the skeleton. If you set the skeleton threshold (in FSLView, the lower of the display range settings) much lower than 0.2, it will extend away towards extremes where there is too much cross-subject variability and where the nonlinear registration has not been able to attain good alignments. Remember the skeleton threshold for the next stage.

<img src="http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/TBSS/UserGuide?action=AttachFile&do=get&target=eg_skel_all_FA.png">

####tbss_4_prestats

The last TBSS script carries out the final steps necessary before you run the voxelwise cross-subject stats. It thresholds the mean FA skeleton image at the chosen threshold - a common value that works well is 0.2 (see above). To run, type:


<pre><code>tbss_4_prestats 0.2</code></pre>
replacing the 0.2 with another value if you need to change it.

The resulting binary skeleton mask defines the set of voxels used in all subsequent processing. Next a "distance map" is created from the skeleton mask. This is used in the projection of FA onto the skeleton (see the TBSS paper for more detail). Finally, the script takes the 4D all_FA image (containing all subjects' aligned FA data) and, for each "timepoint" (i.e., subject ID), projects the FA data onto the mean FA skeleton. This results in a 4D image file containing the (projected) skeletonised FA data. It is this file that you will feed into voxelwise statistics in the next section.


####voxelwise statistics on the skeletonised FA data

The previous step resulted in the 4D skeletonised FA image all_FA_skeletonised (in the stats subdirectory). It is this that you now feed into voxelwise statistics, that, for example, tells you which FA skeleton voxels are significantly different between two groups of subjects.

One recommended way of doing the stats is to use the randomise tool. For more detail see the randomise manual. Before running randomise you will need to generate a design matrix file, e.g., design.mat and contrasts file, e.g., design.con. You can use the script design_ttest2 in the simple case of a two-group comparison. Alternatively you can use the Glm GUI to generate these design matrix and contrast files. Note that the order of the entries (rows) in your design matrix must match the alphabetical order of your original FA images, as that determines the order of the aligned FA images in the final 4D file all_FA_skeletonised; check this with:


<pre><code>cd FA
imglob *_FA.*</code></pre>
We recommend using the TFCE (Threshold-Free Cluster Enhancement) option in randomise. This is somewhat similar to cluster-based thresholding, but generally more robust and avoids the need for the arbitrary initial cluster-forming threshold. To use this on TBSS-preprocessed data, add the --T2 option to randomise.

So, say you have 7 controls (with original filenames CON_001_dti_FA.nii.gz etc.) and 11 patients (with original filenames PAT_001_dti_FA.nii.gz etc.). You can generate design files and run voxelwise statistics and inference using randomisation, including cluster-based thresholding, using:


<pre><code>cd ../stats
design_ttest2 design 7 11

randomise -i all_FA_skeletonised -o tbss -m mean_FA_skeleton_mask -d design.mat -t design.con -n 500 --T2

fslview $FSLDIR/data/standard/MNI152_T1_1mm mean_FA_skeleton -l Green -b 0.2,0.8 tbss_tstat1 -l Red-Yellow -b 3,6 tbss_tstat2 -l Blue-Lightblue -b 3,6</code></pre>

In this case, contrast 1 gives the control>patient test and contrast 2 gives the control<patient test. The raw (unthresholded) tstat images are tbss_tstat1 and tbss_tstat2 respectively. The TFCE p-value images (fully corrected for multiple comparisons across space) are tbss_tfce_corrp_tstat1 and tbss_tfce_corrp_tstat2 (note, these are actually 1-p for convenience of display, so thresholding at .95 gives significant clusters).




####Using non-FA Images in TBSS

It is straightforward to apply TBSS to other diffusion-derived data than FA images. For example, you may be interested in how MD (mean diffusivity) or the first diffusion tensor eigenvalue varies between different subjects.

To achieve this we recommend using the FA images to achieve the nonlinear registration and skeletonisation stages, and also to estimate the projection vectors from each individual subject onto the mean FA skeleton. The nonlinear warps and skeleton projection can then also be applied to other images such as the second eigenvalue. The following instructions assume that you want to run TBSS on the second eigenvalue, named L2 by dtifit:

* Run the full TBSS analysis (see all steps above) on your FA data.
* Create a new directory called L2 (or any other name) in your TBSS analysis directory (the one that contains the existing origdata, FA and stats directories from the FA analysis). Type: mkdir L2
* Copy your L2 images into this new directory, making sure that they are named exactly the same as the original FA images were (look in origdata to check the original names - and keep them exactly the same, even if they include FA, which can be confusing; e.g. if there is an image origdata/subj005_FA.nii.gz then you need an image L2/subj005_FA.nii.gz and this file should contain the L2 data, even though it has FA in the name).
* Now, making sure that you are in your top working TBSS directory (the one that now contains FA, stats and L2 subdirectories) and run the tbss_non_FA script, telling it that the alternate data is called L2. This will apply the original nonlinear registration to the L2 data, merge all subjects' warped L2 data into a 4D file stats/all_L2, project this onto the original mean FA skeleton (using the original FA data to find the projection vectors), resulting in the 4D projected data stats/all_L2_skeletonised. Run: tbss_non_FA L2
* You can now run voxelwise stats on the projected 4D data all_L2_skeletonised in the same manner as described above.

####Using crossing-fibre measures

If you use this option, please site this paper: S. Jbabdi, T. E. Behrens, and S. M. Smith. Crossing fibres in tract-based spatial statistics. NeuroImage, 49:249-256, 2010.

The interpretation of FA changes in crossing-fibre regions can be ambiguous. One solution is to use models that incorporate fibre-specific measurements, i.e. two or more measurements that relate to two or more fibre orientations within each voxel. For example, you may use partial volume fraction estimates from bedpostX instead of FA at each voxel. If you estimate orientations x1 and x2, together with their partial volume fractions f1 and f2, you may want to use f1 and f2 instead of FA which should increase the interpretability of the results in crossing fibre areas.

When using measurements that are orientation specific, it is not sufficient to use tbss_non_FA, because we need to ensure that a fibre orientation x1 in a subject corresponds to the same fibre population across all subjects. The ouput of bedpostX may be such that x1 relates to a given pathway (say cortico-spinal tract) in a subset of the subjects, but to another tract (e.g. SLF) in other subjects. I.e. x1 and x2 are swapped between those two subsets of subjects.

TBSS deals with these situations automatically, via the use of the tbss_x script. As for tbss_non_FA, you can use FA for the nonlinear registration and skeletonisation stages, and also to estimate the projection vectors from each individual subject onto the mean FA skeleton. The nonlinear warps and skeleton projection can then be applied to crossing fibre measurements. The following instructions assume that you are using bedpostX estimates of fibres orientations (dyads1,dyads2,etc.) and related partial volumes (f1,f2,etc). You can replace these with any other orientation-dependent measurements (e.g. using another diffusion model).

* Run the full TBSS analysis (see all steps above) on your FA data. * Create one directory per orientation and per measurement in your TBSS analysis directory (the one that contains the existing origdata, FA and stats directories from the FA analysis):


<pre><code>mkdir F1 F2 D1 D2</code></pre>
* Copy your images into this new directory, making sure that they are named exactly the same as the original FA images were (look in origdata to check the original names). For example, if you are using bedpostX outputs, copy the dyads (dyads1,dyads2,etc.) onto D1,D2,etc., and the partial volumes (mean_f1samples, mean_f2samples,etc.) onto F1,F2,etc. * Now, making sure that you are in your top working TBSS directory (the one that contains FA, stats and origdata subdirectories) and run the tbss_x script:


<pre><code>tbss_x F1 F2 D1 D2</code></pre>
This will apply the original nonlinear registration to the data in F* and D*, then perform a series of operations that ensure that F* will be associated with the same orientations across subjects. * You can now run voxelwise stats on the output 4D data all_F*_x_skeletonised as described above.


####Testing left vs. right in TBSS

In order to test skeleton voxels on the left of the brain vs. those on the right (i.e., testing for asymmetries in diffusion characteristics), it is necessary to derive (and project onto) a slightly different mean FA skeleton - one that is symmetric. Obviously this skeleton needs to be restricted to only those parts of the "original" skeleton that are already sufficiently close to being symmetric - i.e. where there is reasonable correspondence in general tract structure between left and right in the brain.

To run such analysis, first complete the above TBSS processing steps 1-4. Then cd into the stats subdirectory in your TBSS analysis. Use the script tbss_sym; if you want to apply the symmetry analysis to the FA data, type


<pre><code>tbss_sym FA</code></pre>
or if you want to apply it to other diffusion data (that has previously been processed using tbss_non_FA), such as MO, type


<pre><code>tbss_sym MO</code></pre>
The script first generates the symmetric mean-FA image and derived skeleton, mask and distance map, unless these have already been generated by a previous run of tbss_sym. The rest of this paragraph can be ignored if you are not interested in the details of how the symmetric skeleton is derived. First, the original (asymmetric) skeleton is dilated (thickened) by one voxel, and saved to a temporary file. Next, the symmetrised mean-FA image mean_FA_symmetrised is generated, just by flipping and averaging mean_FA. This is then skeletonised to generate the initial symmetric skeleton. This skeleton is masked by the dilated original skeleton - this step ensures that only skeleton structures that are already close to being symmetric in the original data are used in this left-right analysis. Finally, to be absolutely sure the final skeleton is exactly symmetric, it is flipped and masked with the non-flipped version, creating a symmetrised skeleton mean_FA_symmetrised_skeleton. A thresholded skeleton mask image and derived distance map are then derived from this.

Now, the 4D prealigned data all_FA (or non-FA equivalent, if requested) is projected onto the symmetrised skeleton, resulting in 4D file all_FA_symmetrised_skeletonised (etc.). This could now be left-right tested. However, To make this easy, two final steps are carried out: 1) The 4D dataset is left-right flipped, and the latter subtracted from the former. 2) The left half of this dataset (corresponding to the "right" side of standard space) is then zeroed, as the same information (inverted) is present on the left and right sides of the image. This results in 4D image all_FA_skeletonised_left_minus_right.

Therefore the simplest possible analysis with randomise is just to test whether the mean of this data is significantly greater than zero, using the -1 option instead of entering a design matrix and contrast file. This tests for L>R. To test for R>L, either invert the data and re-run randomise, or create an appropriate design matrix and contrast file.

<img src="http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/TBSS/UserGuide?action=AttachFile&do=get&target=stats_eg.png">

####Displaying TBSS Results

stats_eg.png In this section we give some hints on how to present TBSS results. You might want to use either the MNI152 background image or your mean_FA study-specific image - or you may want to show TBSS results on top of both. If you use your mean_FA image as the background, make sure that you set a good display range, for example 0:0.6. For the rest of this section we'll assume that you want to show results on top of the MNI152 image and hence start by loading MNI152 into FSLView.

You will probably next want to load the mean_FA_skeleton image on top of your background image, to show where the skeleton was estimated, and which standard-space voxels were tested in the multi-subjects statistics. Load mean_FA_skeleton into FSLView and set its display range correctly. The lower threshold must be set to the threshold that you used in the TBSS analysis, for example 0.2. The upper level should probably be set to something like 0.7, so that you can see variation in mean FA values within the skeleton. You probably want to change the colourmap, for example to Green, and increase the transparency (with the transparency slider) so that when you load the stats image in, it is easier to see.

Finally, load the stats image in. If you have used TFCE-based testing in randomise, the raw t-statistic image will be named something like tbss_tstat1 (which you could view to see raw tstats before significance testing), but the image you probably want is tbss_tfce_corrp_tstat1, which is the corrected p-value image (actually the values in this image are 1-p for ease of display, so that bigger is "better"). Load this into FSLView, set a colourmap such as Red-Yellow, and set the display range to something like 0.95:1, which corresponds to thresholding the results at p<0.05.

All of the above (apart from setting the skeleton transparency, which has to be done by hand in the GUI) can be carried out with a single command (see first example image):


<pre><code>fslview $FSLDIR/data/standard/MNI152_T1_1mm mean_FA_skeleton -l Green -b 0.2,0.7 tbss_tfce_corrp_tstat1 -l Red-Yellow -b 0.95,1</code></pre>
stats_eg2.png Alternatively, although showing the stats results on the TBSS skeleton is a true representation of the actual analysis carried out, some people find it easier to visualise the results if the skeletonised results are "thickened" somewhat. In order to make such a presentation easy, there is a script tbss_fill, which thickens the thresholded stats image, filling it out into the local "tracts" seen in mean_FA. For example, to apply this to the same example as above and then view in FSLView on top of the mean_FA image, run:


<pre><code>tbss_fill tbss_tfce_corrp_tstat1 0.95 mean_FA tbss_fill
fslview mean_FA -b 0,0.6 mean_FA_skeleton -l Green -b 0.2,0.7 tbss_fill -l Red-Yellow</code></pre>

<img src="http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/TBSS/UserGuide?action=AttachFile&do=get&target=stats_eg2.png">

####Transforming TBSS results back to native space

It is possible to take one or more voxels on the mean FA skeleton and show where, in each subject's FA image, those voxels originally came from. This "back projection" is composed of one or two steps. First, the skeleton voxel is projected back from its position on the skeleton to the nearby position at the centre of the nearest tract in the subject's FA image in standard space (i.e., after the FA image had been nonlinearly registered to the target image). Second, this point can then be "inverse warped" back into the subject's native space, by inverting the nonlinear registration that was originally applied. The whole process can be viewed thus:

There are two obvious reasons why you might want to back-project a skeleton-space voxel (or voxels). The first is to confirm that a given skeleton point was derived from the correct tract-centre points in all subjects. This can generally be achieve by just taking the first of the back projection steps, from the skeleton to the space of the all_FA data, and viewing the back-projected points on top of all_FA (looking at each subject, i.e., timepoint, separately). The second is to take a skeleton-space voxel or set of voxels (for example, a skeleton-space blob that is significantly different between the two groups of subjects in the study) back to the space of the original subjects' data, in order to run tractography for each subject, with that result forming the tractography seed mask.

<img src="http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/TBSS/UserGuide?action=AttachFile&do=get&target=flowchart.png">

#####Option 1: just deproject <skeleton-space-input-image> onto the space of each nonlinearly registered subject in all_FA

For an example of confirming the final tract-centre projection stage, first form an image containing significant voxels from a previous randomise-based statistical analysis of the skeletonised data:


<pre><code>fslmaths tbss_tfce_corrp_tstat1 -thr 0.95 grot
tbss_deproject grot 1
fslview all_FA grot_to_all_FA -l Red-Yellow</code></pre>
The 1 tells the script to just apply stage 1 of the de-projections, i.e. from skeleton space to the all_FA space. This lets you view the interesting skeleton-space voxels in the space of each subject's standard space (nonlinearly registered) FA image - as you move through the timepoints in FSLView you see each subject in turn.

#####Option 2: do the first step and also invert the nonlinear warping, to get back to subjects' native space

For an example of complete back-projection to the space of the native data in FA, run the following, first making sure you run the command from within the stats directory. The 2 flag tells the script to take the back-projection back to the native space:


<pre><code>tbss_deproject grot 2</code></pre>
This results in a separate output for each subject in the stats directory. For example, if your first subject was called subject_001, you would view the results (if you are in the stats directory), with:


<pre><code>fslview ../FA/subject_001_FA subject_001_FA_grot -l Red-Yellow</code></pre>
This is probably the space you want to transform grot into if you are going to use the result to seed tractography analysis.

Finally, you can add the -n flag to the end of the tbss_deproject command to tell the transformations of your original skeleton voxels to keep the exact values in your input data (rather than allowing them to be smoothly interpolated) when applying the reverse transformations. For example, if your skeleton-space image that you feed into tbss_deproject contains mask index numbers 1, 2 and 3 at different mask locations on the skeleton, you would use the -n option, and then the output images from tbss_deproject will contain the same set of index numbers without any alteration cause by the interpolations. Note that this will take a long time to run if you have lots of distinct integer values in the input image, as the deprojections are run separately for every different index number! If your data has continuous values (e.g. Z-statistic or p-values) you should not use the -n option.
