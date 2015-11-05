.. mrdiffusion documentation master file, created by
   sphinx-quickstart on Mon Oct 26 10:48:55 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

----------
User Guide
----------

.. meta::
   :description: User Guide of MR.Diffusion

.. toctree::
   :maxdepth: 3
   :local:


Eddy and motion correction
==========================

During the MRI scanning, many factors can cause magnetic field inhomogeneity, including changing magnetic fields from the imaging gradients and the radiofrequency (RF) coils and yielded biological effects. These effects usually degrade the imaging quality, resulting in artifacts including shearing and blurring (http://mri-q.com/eddy-current-problems.html). Another type of artifacts is caused by head motion. Most of the artifacts described above could be amended by post-processing. In the eddy correction module, we apply a rigid registration method to address most of the shearing and motion artifacts. For the blurring artifacts, it still needs further investigation. Additional solutions would be added to reduce the magnetic field inhomogeneity by field mapping.

.. code-block:: bash

 ccm@:bin$ ./bneddy -h
 bneddy: Head Motion Correction. 
    -i                                        Input file.
    -o                                        Output file.
    -ref             0                       Indicating which frame is the reference.

Brain stripping
===============

This module is to strip the brain skull and extract the brain tissue, including gray matter, white matter, cerebrospinal fluid (CSF) and cerebellum. It largely benefits the following processing and analysis, offering better registration/alignment results and reducing computational time by excluding non-brain tissue. Therefore, although this step is not compulsory, we strongly recommend enforcing it. This module is adapted from the FSL/BET functions (http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/BET) for its excellent efficacy and efficiency. 

.. code-block:: bash

 ccm@:bin$ ./bet2 -h
 Part of FSL (build 504)
 BET (Brain Extraction Tool) v2.1 - FMRIB Analysis Group, Oxford
 Usage: 
 bet2 <input_fileroot> <output_fileroot> [options]
 Optional arguments (You may optionally specify one or more of):
	-o,--outline	generate brain surface outline overlaid onto original image
	-m,--mask <m>	generate binary brain mask
	-s,--skull	generate approximate skull image
	-n,--nooutput	don't generate segmented brain image output
	-f <f>		fractional intensity threshold (0->1); default=0.5; smaller values give larger brain outline estimates
	-g <g>		vertical gradient in fractional intensity threshold (-1->1); default=0; positive values give larger brain outline at bottom, smaller at top
	-r,--radius <r>	head radius (mm not voxels); initial surface sphere is set to half of this
	-w,--smooth <r>	smoothness factor; default=1; values smaller than 1 produce more detailed brain surface, values larger than one produce smoother, less detailed surface
	-c <x y z>	centre-of-gravity (voxels not mm) of initial mesh surface.
	-t,--threshold	-apply thresholding to segmented brain image and mask
	-e,--mesh	generates brain surface as mesh in vtk format
	-v,--verbose	switch on diagnostic messages
	-h,--help	displays this help, then exits


Reconstruction of the diffusion model
=====================================

The reconstruction for diffusion model within pixels is one of the key topics in diffusion MRI research and it is also one of key modules of the software. At the current stage, we have integrated three modeling methods: one is the traditional Gaussian model (commonly known as DTI, diffusion tensor imaging), and the other two are for high angular resolution diffusion imaging (HARDI). For more detailed information please refer to our review paper [11,13]_.

DTI reconstruction
------------------

The classical diffusion gradient sequence used in dMRI is the pulsed gradient spin-echo (PGSE) sequence proposed by Stejskal and Tanner. This sequence has 90o and 180o gradient pulses with duration time δ and separation time Δ. To eliminate the dependence of spin density, we need at least two measurements of diffusion weighted imaging (DWI) signals, i.e. S(b) with the diffusion weighting factor b in Eq. (1) introduced by Le Bihan etc, and S(0) with b = 0 which is the baseline signal without any gradient. 

.. raw:: html

 $$ \begin{equation}\tag{1}
 b=\gamma^2 \delta^2 (\Delta-\frac{\delta}{3}){||\bf{G}||}^2
 \end{equation} $$

In the b-value in Eq. (1), $\\gamma$ is the proton gyromagnetic ratio, $\\bf{G}=||\\bf{G}||\\bf{u}$ is the diffusion sensitizing gradient pulse with norm $||{\\bf G}||$ and direction ${\\bf u}$. $\\tau=\\Delta-\\frac{1}{3}\\delta$ is normally used to describe the effective diffusion time. Using the PGSE sequence with S(b), the diffusion weighted signal attenuation E(b) is given by Stejskal-Tanner equation,

.. raw:: html

 $$ \begin{equation}\tag{2}
 E(b)=\frac{S({b})}{S(0)}=\exp(-{b}D)
 \end{equation} $$

where D is known as the apparent diffusion coefﬁcient (ADC) which reﬂects the property of surrounding tissues. Note that in general ADC D is also dependent on G in a complex way. However, free diffusion in DTI assumes D is only dependent on the direction of G, i.e. . The early works in dMRI reported that ADC D is dependent on gradient direction u and used two or three DWI images in different directions to detect the properties of tissues. Then Basser et al. introduced diffusion tensor [12]_ to represent ADC as $D(\\bf{u}) = {\\bf u^{T}}{\\bf D}\\bf{u}$, where ${\\bf D}$ is called as the diffusion tensor, which is a 3 × 3 symmetric positive deﬁnite matrix independent of u. This method is called as diffusion tensor imaging (DTI) and is the most common method nowadays in dMRI technique. In DTI, the signal E(b) is represented as 

.. raw:: html

 $$ \begin{equation}\tag{3}
 E(b)=\exp(-b{\bf u^{T}}{\bf D}\bf{u})
 \end{equation} $$


.. figure:: images/scalarmaps.png
   :width: 800 px
   :alt: Tensor field and the scalar maps
   :align: center

   Figure 4. Tensor field and the scalar maps estimated from a monkey data with b = 1500s/mm2.

The diffusion tensor D can be estimated from measured diffusion signal samples  through a simple least square method or weighted least square method [12]_, or more complex methods that consider positive deﬁnite constraint or Rician noise. If single b-value is used, the optimal b-value for tensor estimation was reported to in the range of (0.7, 1.5) × 10−3 s/mm2, and normally about twenty DWI images are used in DTI in clinical study. ome useful indices can be obtained from tensor D. The most important three indices are fractional anisotropy (FA), mean diffusivity (MD) and relative anisotropy (RA) deﬁned as 

.. raw:: html

 $$ \begin{equation}\tag{4}
 {\rm{FA}}=\frac{\sqrt{3}||{\rm{D}}-\frac{1}{3}Trace({\rm{D}}) I ||}{\sqrt{2}||{\rm{D}}||}
 =\sqrt{\frac{3}{2}}\sqrt{\frac{(\lambda_1-\bar{\lambda})^2+(\lambda_2-\bar{\lambda})^2+(\lambda_3-\bar{ \lambda})^2}{\lambda_{1}^{2}+\lambda_{2}^{2}+\lambda_{3}^{2}}}
 \end{equation} $$

.. raw:: html
 
 $$ \begin{equation}\tag{5}
 {\rm{MD}}=\bar{\lambda}=\frac{1}{3}Trace({\rm{D}})=\frac{\lambda_1+\lambda_2+\lambda_3}{3}
 \end{equation} $$ 

.. raw:: html
 
 $$ \begin{equation}\tag{6}
 {\rm{RA}}=\sqrt{\frac{(\lambda_1-\bar{\lambda})^2+(\lambda_2-\bar{\lambda})^2+(\lambda_3-\bar{\lambda})^2}{3\bar{\lambda}}}
 \end{equation} $$ 

where,  are the three eigenvalues of D and  is the mean eigenvalue. MD and FA have been used in many clinical applications. For example, MD is known to be useful in stroke study. For more detailed information please refer to our review paper [13]_.

.. code-block:: bash

 ccm@:bin$ ./bndti_estimate -h
 bndti_estimate: Diffusion Tensors Estimation.
 MR. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Oct  9 2015, 19:26:40)
 general arguments
    -d                                Input DWI Data, in NIFTI/Analyze format (4D)
    -g                                Gradients direction file
    -b                                b value file
    -m                                Brain mask : filein mask | OPTIONAL
    -o                                Result DTI : fileout DTI
    -tensor          0                Save tensor : 0 - No; 1 - Yes; (Default: 0)
    -eig             0                Save eigenvalues and eigenvectors : 0 - No; 1 - Yes; (Default: 0)

SPFI reconstruction
-------------------

It was proposed that the SPFI method has more powerful capability to identify the tangling fibers [8]_. In SPFI [8]_, the diffusion signal  is represented by spherical polar Fourier (SPF) basis functions in Eq. 7. 

The SPF basis denoted by  is a 3D orthonormal basis with spherical harmonics in the spherical portion and the Gaussian-Laguerre function in the radial portion. Furthermore, Cheng and his colleagues proposed a uniform analytical solution to transform the coefficients  of  to the coefficients  of ODF (orientation distribution function) represented by the spherical harmonics basis, as in Eq. 8. 


It is a model-free, regularized, fast and robust reconstruction method which can be performed with single-shell or multiple-shell HARDI data to estimate the ODF proposed by Wedeen et al. [14]_. The implementation of analytical SPFI includes two independent steps. The first estimates the coefficients of  with least squares, and the second transforms the coefficients of  to the coefficients of ODF.

.. code-block:: bash

  ccm@:bin$ bnhardi_ODF_estimate -h
  bnhardi_ODF_estimate: Orientation Distribution Function Estimation (SPFI method).
  MR. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
  (Jul 15 2015, 11:50:20)
    -d                                        dwi data
    -b                                        text file contains b-value
    -g                                        text file contains grad direction
    -m                                        Brain mask.
    -o                                        output odf file
    -scale           -1                       if not given, a suggested scheme is used
    -tau             0.02533                  tar value to calculate the true pdf.
    -b0_w            1                        b0 weight for the least square estimate
    -b0_analytical   1                        true: analytical
    -is_assemlal     true                     true: assemlal basis; false: cheng basis
    -outGFA          false                    output GFA: true or false
    -rdis            0.015                    r value for pdf.
    -sh              4                        order of spherical harmonics
    -ra              1                        order of radial part
    -lambda_sh       0                        regualrization parameter for sh basis
    -lambda_ra       0                        regualrization parameter for ra basis

CSD reconstruction
------------------

The CSD method was proposed by Tournier et al. [9]_, which expresses the diffusion signal as in Eq. 9,

where  is called the fiber orientation density function (fODF), which needs to be estimated, and  is the response function, which is the typical signal generated from one fiber. The response function can be directly estimated from diffusion weighted image (DWI) data by measuring the diffusion profile in the voxels with the highest fractional anisotropy values, indicating that a single coherently oriented fiber population is contained in these voxels. When the response function is obtained, we can utilize the deconvolution of  from  to estimate the fiber ODF. The computation of the fiber ODF was carried out using the software MRtrix (J-D Tournier, Brain Research Institute, Melbourne, Australia, http://www.brain.org.au/software/). We thank Dr. Jacques-Donald Tournier for sharing MATLAB code of the CSD method, which inspired a quick C/C++ implementation.

.. code-block:: bash

 ccm@:bin$ ./bnhardi_FOD_estimate -h
 bnhardi_FOD_estimate: Constraind Spherical Deconvolution (CSD) based HARDI reconstruciton.
 MR. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Jul 15 2015, 11:50:20)
 general arguments
    -d                                        Input DWI Data, in NIFTI/Analyze format (4D)
    -g                                        Gradients direction file
    -b                                        b value file
    -m                                        Brain mask : filein mask | OPTIONAL
    -outFA           1                        Whether to output the FA of DTI
    -o                                        Result CSD Estimate (.nii.gz)
    -lmax            8                        6/8/10, Max order of the adopted harmonical base
    -fa              [0.75,0.95]              The FA thesshold considered as single fiber
    -erode           -1                       The unit is voxel: Remove the garbage near the boundary of FA image, for better estimating response function
    -nIter           50                       Max iteration number before aborting
    -lambda          1                        The regularization weight for optimization
    -tau             0.1                      The threshold on the FOD amplitude used to identify negative lobes
    -hr             300                      300/1000/5000. The later get more acurate estimation while more time consuming, so use the first one unless your computer is powerful !


Fiber tracking and attributes extraction
========================================

Fiber tracking is a critical way to construct the anatomical connectivity matrix. For the tracking based on tensors from DTI, an intuitive way is to link the neighboring voxels following their main directions (e.g. V1 in the eigenvector of the DTI) given a set of some stop criteria, such as maximum bending angle of the curve and minimum FA value, which is to ensure the target voxel is indeed white matter microstructure. This is the so called deterministic streamline tractography [15]_, as illustrated in Figure 5.

.. figure:: images/tractography.png
   :width: 800 px
   :alt: Illustration for deterministic streamline tractography
   :align: center

   Figure 5. Illustration for deterministic streamline tractography.

.. code-block:: bash

 ccm@:bin$ ./bndti_tracking -h
 bndti_tracking: DTI Deterministic Fibertracking.
 MR. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Oct  9 2015, 19:26:46)
    -d                                        Input DTI data.
    -m                                        Mask Image.
    -s                                        Seeds Image.
    -fa                                       FA Image.
    -ft              0.1                      FA Threshold
    -at              45                       Angular Threshold.
    -sl              0.5                      Step Length (Voxel).
    -min             10                       Threshold the fiber (mm). (remove fibers shorter than the number)
    -max             5000                     Upper-threshold the fiber (mm). (remove fibers longer than the number)
    -o                                        Output Fibers Filename (.fiber file).

The tracking module in the software for HARDI estimation is similar to the streamline method as described above. It should be kept in mind that, for HARDI estimation, usually there are more than one main direction (which is what we desired since it possibly identifies tangling fibers), so we should consider these kissing/branching cases. Meanwhile, since there is no explicit dominant directions for each voxel, searching algorithm should be applied to locate the main directions. The searching should run for each voxel, so it is not as fast as the traditional DTI tracking.

.. code-block:: bash

 ccm@:bin$ ./bnhardi_tracking -h
 bnhardi_tracking: HARDI Deterministic Fibertracking.
 Mr. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Oct  9 2015, 19:26:46)
    -d                                        HARDI spherical harmonic image.
    -a                                        Anisotropy image.
    -m                                        Mask file.
    -s                                        Seeds file.
    -o                                        Fiber file (.fiber file).
    -sl              0.5                      Step length (Voxel): float
    -ft              0.1                      FA threshold: float
    -at              75                       Angle threshold: float
    -min             10                       Threshold the fiber (mm). (remove fibers shorter than the number)
    -max             5000                    Upper-threshold the fiber (mm). (remove fibers longer than the number)

Visualization for various images
================================

The software provides a variety of entries for visualizing different data types, including 3D/4D image, Tensor/ODF/FOD image and white mater fibers. The views of different data could be superimposed for a precise anatomical localization. It should be noted that, if one uses the GUI, these view functions could be called in each processing steps for inspecting the results. 

Figure 6. Illustrations for the capability of the software to show many types of images.

.. code-block:: bash

 ccm@:bin$ ./bnviewer -h
 This program is the GUI frontend that displays and performs data reconstruction and fiber tracking on diffusion MR images, which have been developed by the teams of Brainnetome Center, CASIA.
 basic usage: 
   bnviewer [[-volume] DTI_FA.nii.gz] 
            [-roi ROI/roi_cc_top.nii.gz] 
            [-fiber ROI/roi_cc_top.fiber] 
            [-tensor DTI.nii.gz]/[-odf HARDI.nii.gz]
            [-atlas DTI.nii.gz]
 options:  
   -help                  show this help
   -volume  .nii.gz       set input background data
   -roi     .nii.gz       set input ROI data
   -fiber   .fiber/.trk   set input fiber data
   -tensor  .nii.gz       set input DTI data (conflict with -odf args)
   -odf     .nii.gz       set input ODF/FOD data (conflict with -tensor args)

Several useful tools
====================

Image registration
------------------

This module is implemented by elastix [10]_, which integrates a collection of functions from ITK [1]_. In our software, the registration module is customized to auto-configure the parameter settings between different image modalities, e.g., two DWI images (for eddy current correction in current version), standard space and T1 space (for mapping ROIs to the individual space), DWI space and standard space (for statistical comparisons across subjects). This module contains two separate submodules, <code>elastix</code> for computing the transform matrix and <code>transformix</code> for applying the existing transform matrix to an image.

.. code-block:: bash

 ccm@:bin$ ./elastix -h
 elastix version: 4.700
 elastix registers a moving image to a fixed image.
 The registration-process is specified in the parameter file.
  --help, -h displays this message and exit
  --version  output version information and exit
 Call elastix from the command line with mandatory arguments:
  -f        fixed image
  -m        moving image
  -out      output directory
  -p        parameter file, elastix handles 1 or more "-p"
 Optional extra commands:
  -fMask    mask for fixed image
  -mMask    mask for moving image
  -t0       parameter file for initial transform
  -priority set the process priority to high, abovenormal, normal (default),
            belownormal, or idle (Windows only option)
  -threads  set the maximum number of threads of elastix

The parameter-file must contain all the information necessary for elastix to runproperly. That includes which metric to use, which optimizer, which transform, etc. It must also contain information specific for the metric, optimizer, transform, etc. For a usable parameter-file, see the website.

Need further help?
******************

Check the website http://elastix.isi.uu.nl, or mail elastix@bigr.nl.

.. code-block:: bash

 ccm@:bin$ ./transformix -h
 transformix version: 4.700
 transformix applies a transform on an input image and/or generates a deformation field.
 The transform is specified in the transform-parameter file.
  --help, -h displays this message and exit
  --version  output version information and exit

Call transformix from the command line with mandatory arguments:

-out      output directory
-tp       transform-parameter file, only 1

Optional extra commands:

-in        input image to deform
-def       file containing input-image points; the point are transformed
           according to the specified transform-parameter file
           use "-def all" to transform all points from the input-image, which
           effectively generates a deformation field.
-jac       use "-jac all" to generate an image with the determinant of the
           spatial Jacobian
-jacmat    use "-jacmat all" to generate an image with the spatial Jacobian
           matrix at each voxel
-priority  set the process priority to high, abovenormal, normal (default),
           belownormal, or idle (Windows only option)
-threads   set the maximum number of threads of transformix

At least one of the options "-in", "-def", "-jac", or "-jacmat" should be given.

The transform-parameter file must contain all the information necessary for transformix to run properly. That includes which transform to use, with which parameters, etc. For a usable transform-parameter file, run elastix, and inspect the output file "TransformParameters.0.txt".

Image calculation and ROI generation
------------------------------------

The module bncalc provides simple image calculations, such as add/minus/multiply/divide operations. Meanwhile, it can also generate user-defined ROIs given the origin and radius in a user-specified image space.

.. code-block:: bash

 ccm@:bin$ ./bncalc -h
 Mr. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 This funciton provide basic process for the input data (NIFTI/Analyze format)
 Usage of bncalc:
    -i       image         The original file you want to manage.
    -add     image/value   Add to the data from the last step.
    -sub     image/value   Subtract data from the last step.
    -mul     image/value   Multiply the data from the last step.
    -div     image/value   Divide the data from the last step.
    -roi     x,y,z,r       Generate a ROI centered at [x,y,z](MNI mm) with radius r,
                           in the input data space.
    -mask    image         Mask the data from last step by this input one. 
                           If this input is a binary, then it is the same as 
                           -mul, otherwise it keep the voxels from the last step 
                           when the new input is nonzero.
    -bin     value         set 1 if >value, otherwise 0. 
    -uthr    value         Set voxel=0 when it>=value. 
    -dthr    value         Set voxel=0 when it<=value. 
    -o       image         output a NIFTI (.nii.gz) file

Fiber manipulation
------------------

After obtaining a large bundle of white matter fibers, you may want to prune the fibers that go through specified locations (ROIs). Here, we provide several tools to manipulate the fiber bundles.
bnfiber_manipulate, to split/merge fiber bundles based on given ROIs.

.. code-block:: bash

 bnfiber_manipulate: Merge or prune fiber bundles based on given ROIs.
 Mr. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Sep 17 2015, 14:47:12)
    -fiber                                    fiber file
    -and                                      AND file: ro1.nii.gz,roi2.nii.gz
    -or                                       OR file: roi1.nii.gz,roi2.nii.gz
    -not                                      NOT file: ro1.nii.gz,roi2.nii.gz
    -o                                        Output file (.fiber file).

bnfiber_end, to cut the fiber bundles given start/stop ROIs, which is useful to get the exact connections between two ROIs in constructing connectivity matrix.

.. code-block:: bash

 ccm@:bin$ ./bnfiber_end -h
 bnfiber_end: Extract fibers which end in the two given rois.Mr. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Sep 17 2015, 14:47:12)
    -fiber                                    fiber file
    -roi1                                     roi1 file
    -roi2                                     roi2 file
    -o                                        Output file (.fiber file)

bnfiber_stats, to extract statistical properties of the fiber bundle, such as mean FA/MD and fiber numbers.

.. code-block:: bash

 ccm@:bin$ ./bnfiber_stats -h
 bnfiber_stats: Show fiber stats.
 Mr. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Sep 17 2015, 14:47:12)
    -fiber                                    Input fiber file, then output mean FA/MD, number of fibers et al.

bnfiber_map, to compute the fiber density map which is used in track density imaging [16]_.

.. code-block:: bash

 ccm@:bin$ ./bnfiber_map -h
 bnfiber_map: Calculate the fiber density according to the reference volume.
 Mr. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Sep 17 2015, 14:47:13)
    -fiber                                    Fiber file
    -ref                                      Reference file (NIfTI)
    -o                                        Output file (NIfTI).
    -nor             1                        Normalize fiber density: 1(yes) or 0(no).

bnmerge/bnsplit, to merge the 3D volumes to 4D or split 4D to 3D volumes.

.. code-block:: bash

 ccm@:bin$ ./bnmerge -h
 bnmerge: Merge 3D NIfTI files  to 4D NIfTI file.
    This program is to merge 3D NIfTI files to 4D NIfTI file. 
    Usage: bnmerge filein1 filein2 ... fileout
    options:     -h           : show this help

.. code-block:: bash

 ccm@:bin$ ./bnsplit -h
 bnsplit: Split 4D volume to 3D volumes.
    This program splits 4D volume to 3D volumes. 
    basic usage: split -i FILE_IN -o FILE_OUT prefix
    options:     -h           : show this help
                 -v LEVEL     : the verbose level to LEVEL

bninfo, to display a short head information of the input image. Supported input image format includes NIFTI/DICOM.

.. code-block:: bash

 ccm@:DWI$ bninfo -h
 bninfo: Show file header information.
 Mr. Diffusion (v1.0), http://www.brainnetome.org/en/brat.html. 
 (Jul 15 2015, 11:50:01)
    -i                                        Nifti/ANALYZE/DICOM file.

.. include:: common.txt

