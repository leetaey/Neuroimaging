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
이 과정은 필수과정은 아니며 각자의 raw data의 상황에 따라서 적용을 할 수도 있고 하지 않을 수도 있다. 일반적으로 T1 data는 1x1x1 데이터를 dwi data는 2x2x2 데이터를 그리고 fMR 데이터는 3x3x3을 권장하곤 하지만 그도 일정한 기준은 아니다. 마찬가지로 orientation의 경우도 정해진 원칙은 없으니 가급적이면 각기 다른 데이터의 orientation을 한 방향으로 일치시켜두는 것은 잠재적인 에러를 줄이는데 도움이 될 수 있다. 일반적인 표기는 Left(negative) to Right(positive), Posterior(neg.) to Anterior(pos.) 그리고 Inferior(neg.) to Superior(pos.) 이다. 

<pre><code>3dresample -dxyz 2.0 2.0 2.0 -orient LPI -input ${i}_float.nii.gz -prefix ${i}_resample.nii.gz</code></pre>

