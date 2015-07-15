#### Prepare your data for the FSL-VBM study

a) Place all your T1-weighted data in your FSL-VBM directory. For instance:

<pre><code>mkdir my_fsl_vbm</code></pre>

Then copy into your FSL-VBM directory all of your subjects' T1 images, giving each subject's T1 image a different name, preferably with each prefix corresponding to each of your group, for example:

<pre><code>CON_2304.nii.gz
CON_2878.nii.gz
CON_3456.nii.gz
CON_4133.nii.gz
CON_4690.nii.gz
PAT_2042.nii.gz
PAT_2280.nii.gz
PAT_2632.nii.gz
PAT_3193.nii.gz
PAT_4134.nii.gz
PAT_5357.nii.gz
PAT_6659.nii.gz</code></pre>

b) If you have more than one group and the number of subjects in each is not the same, choose (at random) among the biggest group(s) the images that you will use to create the study-specific template, with the same number as of the smallest group (in order to create an unbiased template - see below for further explanation). Once you've chosen which T1 images to keep to build the template, put all the selected names of exams in a file called template_list in your FSL-VBM directory.

All your different populations included in this study MUST be represented in the template construction.

For instance, as we have only 5 controls for 7 patients, we have to select 5 patients out of the 7:

<pre><code>for g in CON_2304.nii.gz CON_2878.nii.gz CON_3456.nii.gz CON_4133.nii.gz CON_4690.nii.gz PAT_2042.nii.gz \
PAT_2632.nii.gz PAT_3193.nii.gz PAT_4134.nii.gz PAT_6659.nii.gz; do
echo $g >> template_list
done</code></pre>

c) At this point you should have a quick look at all your data to check that all subjects' structural images are what you expected:

<pre><code>slicesdir `imglob *`</code></pre>

The imglob command lists all of your input images. The slicesdir command takes the list of images and creates a simple web-page containing snapshots for each of the images. Once it has finished running it tells you the name of the web page to open in your web browser, to view the snapshots. Have a careful look.

d) It's a good idea to consider your cross-subject statistical model before you run the FSL-VBM analysis. So you should at this point create your design.mat and design.con in your FSL-VBM directory; see the randomise manual.

WARNING!!! The order of the rows in your design.mat model MUST match the order of your images when doing an 'ls' command in your FSL-VBM directory.

