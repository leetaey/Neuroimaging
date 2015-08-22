### Common Extraction Mask

<pre><code>3dcalc -a aparc_resample.nii.gz -expr 'equals(a,1) + equals(a,4) + equals(a,5) + equals(a,14) + equals(a,15) + equals(a,16) + equals(a,24) + equals(a,26) + equals(a,27) + equals(a,28) + equals(a,29) + equals(a,30) + equals(a,31) + equals(a,32) + equals(a,33) + equals(a,34) + equals(a,35) + equals(a,36) + equals(a,37) + equals(a,38) + equals(a,39) + equals(a,40) + equals(a,43) + equals(a,44) + equals(a,58) + equals(a,59) + equals(a,60) + equals(a,61) + equals(a,62) + equals(a,63) + equals(a,64) + equals(a,65) + equals(a,66) + equals(a,67) + equals(a,68) + equals(a,69) + equals(a,70) + equals(a,71) + equals(a,72) + equals(a,75) + equals(a,76) + equals(a,85)' -prefix roi.ext.nii.gz</code></pre>

