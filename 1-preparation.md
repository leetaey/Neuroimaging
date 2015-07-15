#### Changes center coordinates of header file 
전체 공간에서 정중앙에 이미지가 위치하도록 만드는 과정이다. 추후 registration을 할 때, 각 이미지 파일의 FOV 차이로 인하여 의도하지 않게 이미지가 잘리는 경우를 방지하기 위하여 이미지 헤더파일에 있는 coordinates 정보를 {0, 0, 0} 이 되도록 교정한다. 3dAttribute를 통하여 이미지의 중심이 {0, 0, 0} 에서 얼마나 떨어져 있는지 정보를 알아내어 cc 라는 변수에 입력하고 {x, y, z} 에 역수를 곱해줘서 dx, dy, dz 변수에 저장한 뒤 다시 원본 이미지파일의 헤더에 적용해준다. 

<pre><code>cc=(`3dAttribute -center dwi.nii.gz`)   
dx=`ccalc "-1 * ${cc[0]}"`
dy=`ccalc "-1 * ${cc[1]}"`
dz=`ccalc "-1 * ${cc[2]}"`
3drefit -dxorigin $dx -dyorigin $dy -dzorigin $dz dwi.nii.gz</code></pre>

#### Changes data type from integer to float
기본적으로 정수로 이루어진 이미지포맷은 추후 이미지 변환을 할때 발생하는 소숫점 이하의 정보들을 다 잃어버리게된다. 따라서 이것을 고려하지 않고 그냥 registration을 하게되면 오차의 주 원인이 된다. 한 논문에 의하여 최대 10% 까지 mis-registration 이 발생할 수 있다고 한다. 따라서 이과정을 통하여 이미지 포맷을 integer에서 float으로 미리 변환해두는 것이 중요하다. 단, 이미지 변환 이후에는 데이터 용량이 2배로 늘어나게된다. 

<pre><code>3dcalc -a dwi.nii.gz -expr a -datum float -prefix dwi_float.nii.gz</code></pre>

#### Changes of voxel sizes and orientation (optional)

<pre><code>3dresample -dxyz 2.0 2.0 2.0 -orient RSP -input ${i}_float.nii.gz -prefix ${i}_resample.nii.gz</code></pre>

