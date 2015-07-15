#### Changes center coordinates of header file 

<pre><code>cc=(`3dAttribute -center dwi.nii.gz`)   
dx=`ccalc "-1 * ${cc[0]}"`
dy=`ccalc "-1 * ${cc[1]}"`
dz=`ccalc "-1 * ${cc[2]}"`
3drefit -dxorigin $dx -dyorigin $dy -dzorigin $dz dwi.nii.gz</code></pre>

#### Changes data type from integer to float

<pre><code>3dcalc -a dwi.nii.gz -expr a -datum float -prefix dwi_float.nii.gz</code></pre>

#### Changes of voxel sizes and orientation (optional)

<pre><code>3dresample -dxyz 2.0 2.0 2.0 -orient RSP -input ${i}_float.nii.gz -prefix ${i}_resample.nii.gz</code></pre>

