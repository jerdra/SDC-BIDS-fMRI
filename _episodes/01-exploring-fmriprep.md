
---
title: "Exploring Preprocessed fMRI Data from fMRIPREP"
teaching: 20
exercises: 5
questions:
- "How does fMRIPrep store preprocessed neuroimaging data"
- "How do I access preprocessed neuroimaging data"
objectives:
- "Learn about fMRIPrep derivatives"
- "Understand how preprocessed data is stored and how you can access key files for analysis"
keypoints:
- "fMRIPrep stores preprocessed data in a 'BIDS-like' fashion"
- "You can pull files using pyBIDS much like how you can navigate raw BIDS data"
---

## Exploring Preprocessed fMRI Data from fMRIPREP

BIDS applications such as fMRIPREP output data into a full data structure with strong similarity to BIDS organization principals. In fact, there is a specification for derivatives (outputs derived from) BIDS datasets; although this is a current work in progress, details can be found in: [BIDS Derivatives](https://bids-specification.readthedocs.io/en/latest/06-extensions.html). 

In this tutorial, we'll explore the outputs generated by fMRIPREP and get a handle of how the data is organized from this preprocessing pipeline

Luckily the semi-standardized output for fMRIPrep is organized in such a way that the data is easily accessible using pyBIDS! We'll first show what the full data structure looks like, then we will provide you with methods on how you can pull specific types of outputs using pyBIDS.

### The fMRIPrep Derivative Data Structure

First let's take a quick look at the fMRIPrep data structure:

~~~
tree -L 1 '../data/ds000030/derivatives/fmriprep'
~~~
{: .language-bash}

~~~

../data/ds000030/derivatives/fmriprep/
├── sub-10171
├── sub-10292
├── sub-10365
├── sub-10438
├── sub-10565
├── sub-10788
├── sub-11106
├── sub-11108
├── sub-11122
├── sub-11131
├── sub-50010
├── sub-50035
├── sub-50047
├── sub-50048
├── sub-50052
├── sub-50067
├── sub-50075
├── sub-50077
├── sub-50081
└── sub-50083
~~~
{: .output}

First note that inside the fMRIPrep folder, we have a folder per-subject. Let's take a quick look at a single subject folder:

~~~
tree '../data/ds000030/derivatives/fmriprep/sub-10788'
~~~
{: .language-bash}

~~~
../data/ds000030/derivatives/fmriprep/sub-10788/
├── anat
│   ├── sub-10788_T1w_brainmask.nii.gz
│   ├── sub-10788_T1w_preproc.nii.gz
│   ├── sub-10788_T1w_space-MNI152NLin2009cAsym_brainmask.nii.gz
│   └── sub-10788_T1w_space-MNI152NLin2009cAsym_preproc.nii.gz
└── func
    ├── sub-10788_task-rest_bold_confounds.tsv
    ├── sub-10788_task-rest_bold_space-fsaverage5.L.func.gii
    ├── sub-10788_task-rest_bold_space-fsaverage5.R.func.gii
    ├── sub-10788_task-rest_bold_space-MNI152NLin2009cAsym_brainmask.nii.gz
    ├── sub-10788_task-rest_bold_space-MNI152NLin2009cAsym_preproc.nii.gz
    ├── sub-10788_task-rest_bold_space-T1w_brainmask.nii.gz
    └── sub-10788_task-rest_bold_space-T1w_preproc.nii.gz

2 directories, 11 files
~~~
{: .output}

As you can see above, each subject folder is organized into an <code>anat</code> and <code>func</code> sub-folder. 

Specifically:

- the <code>anat</code> folder contains the preprocessed anatomical data. If multiple T1 files are available (all T1s even across sessions), then these data are merged - you will always have one <code>anat</code> folder under the subject folder
- the <code>func</code> folder contains the preprocessed functional data. All tasks are dumped into the same folder and like the BIDS convention are indicated by the use of their filenames (<code>task-[task_here]</code>)

> This data is single-session, so a session folder is missing here - but with multiple sessions you will see <code>anat</code> and <code>ses-[insert_session_here]</code> folders where each session folder contain a <code>func</code> folder.
{: .callout}

Hopefully you're now convinced that the outputs of fMRIPREP roughly follows BIDS organization principles and is, in fact, quite simple. The filenames themselves give you a full description of what each file is (check the [slides](https://docs.google.com/presentation/d/1er6dQcERL-Yeb5-7A29tJnmqgHNaLpTLXM3e-SmpjDg/edit?usp=sharing) to get an idea of what each file means!

Now let's see how we can pull data in using pyBIDS!

Let's import pyBIDS through the <code>bids</code> module first:

~~~
import bids
~~~
{: .language-python}

We can make a <code>bids.BIDSLayout</code> object as usual by just feeding in the fmriprep directory! However, one caveat is that since the fmriprep outputs are *not really BIDS but BIDS-like*, we have to turn off bids validation:

~~~
layout = bids.BIDSLayout('../data/ds000030/derivatives/fmriprep/',validate=False)
~~~
{: .language-python}

Now that we have a layout object, we can pretend like we're working with a BIDS dataset! Let's try some common commands that you would've used with a BIDS dataset:

First, we'll demonstrate that we can grab a list of pre-processed subjects much like in the way we would grab subjects from a raw BIDS dataset:

~~~
layout.get_subjects()
~~~
{: .language-python}

~~~
['10171',
 '10292',
 '10365',
 '10438',
 '10565',
 '10788',
 '11106',
 '11108',
 '11122',
 '11131',
 '50010',
 '50035',
 '50047',
 '50048',
 '50052',
 '50067',
 '50075',
 '50077',
 '50081',
 '50083']
 ~~~
 {: .output}

 We can also do the same for tasks

 ~~~
 layout.get_tasks()
 ~~~
 {: .language-python}

 ~~~
 ['rest']
 ~~~
 {: .output}

 Now let's try fetching specific files. Similar to how you would fetch BIDS data using pyBIDS, the exact same syntax will work for fMRIPREP derivatives. Let's try pulling just the preprocessed anatomical data. 

Recall that the anatomical folder is named as follows:

~~~
tree '../data/ds000030/derivatives/fmriprep/sub-10788/anat'^
~~~
{: .language-bash}

~~~
../data/ds000030/derivatives/fmriprep/sub-10788/anat
├── sub-10788_T1w_brainmask.nii.gz
├── sub-10788_T1w_preproc.nii.gz
├── sub-10788_T1w_space-MNI152NLin2009cAsym_brainmask.nii.gz
└── sub-10788_T1w_space-MNI152NLin2009cAsym_preproc.nii.gz

0 directories, 4 files
~~~
{: .output}

The file that we're interested in is of form <code>sub-[subject]_T1w_preproc.nii.gz</code>. Now we can construct a pyBIDS call to pull these types of files specifically:

~~~
preproc_T1 = layout.get(datatype='anat',suffix='preproc',extension='.nii.gz')
preproc_T1
~~~

~~~
[<BIDSImageFile filename='/mnt/tigrlab/projects/jjeyachandra/scwg2018_python_neuroimaging/data/ds000030/derivatives/fmriprep/sub-10171/anat/sub-10171_T1w_preproc.nii.gz'>,
 <BIDSImageFile filename='/mnt/tigrlab/projects/jjeyachandra/scwg2018_python_neuroimaging/data/ds000030/derivatives/fmriprep/sub-10171/anat/sub-10171_T1w_space-MNI152NLin2009cAsym_preproc.nii.gz'>,
 <BIDSImageFile filename='/mnt/tigrlab/projects/jjeyachandra/scwg2018_python_neuroimaging/data/ds000030/derivatives/fmriprep/sub-10292/anat/sub-10292_T1w_preproc.nii.gz'>,
 ...
~~~
{: .output}

Note that we also pulled in MNI152NLin2009cAsym_preproc.nii.gz data as well. This is data that has been transformed into MNI152NLin2009cAsym template space. We can pull this data out by further specifying our <code>layout.get</code> using the <code>space</code> argument:

~~~
mni_preproc_T1 = layout.get(datatype='anat',suffix='preproc',extension='.nii.gz',space='MNI152NLin2009cAsym')
mni_preproc_T1
~~~
{: .language-python}

~~~
[<BIDSImageFile filename='/mnt/tigrlab/projects/jjeyachandra/scwg2018_python_neuroimaging/data/ds000030/derivatives/fmriprep/sub-10171/anat/sub-10171_T1w_space-MNI152NLin2009cAsym_preproc.nii.gz'>,
 <BIDSImageFile filename='/mnt/tigrlab/projects/jjeyachandra/scwg2018_python_neuroimaging/data/ds000030/derivatives/fmriprep/sub-10292/anat/sub-10292_T1w_space-MNI152NLin2009cAsym_preproc.nii.gz'>,
 ...
~~~
{: .output}

What if we wanted to pull out the data in T1 "native space" (it really is a template space, since it is merged T1s)? Unfortunately for *this specific version of fMRIPREP* this isn't directly possible using <code>layout.get</code>. Instead we'll use a bit of python magic to pull the data that we want:

>> In newer versions of fMRIPREP <code>space</code> is included in the native T1w file filename as <code>space-T1w</code> - in this case you can pull the data by using <code>layout.get(..., space='T1w')</code>
{: .callout}

~~~
native_preproc_T1 = [f for f in preproc_T1 if f not in mni_preproc_T1]
native_preproc_T1
~~~
{: .language-python}


Similarily fMRI data can be pulled by specifying <code>datatype='func'</code> and using the <code>suffix</code> argument as appropriate:

> ## Exercise 1
> 1. Get the list of **all** preprocessed functional data
> 2. Get the list of functional data in MNI152NLin2009cAsym space
> 3. Get the list of functional data in T1w space (native)
> >  ## Solution
> > *All the functional data*
> > ~~~
> > func_data = layout.get(datatype='func', suffix='preproc')
> > ~~~
> > {: .language-python}
> > 
> > *MNI152NLin2009cAsym Functional Data*
> > ~~~
> > mni_func_data = layout.get(datatype='func', suffix='preproc', space='MNI152NLin2009cAsym')
> > mni_func_data
> > ~~~
> > {: .language-python}
> > 
> > *Native/T1w space functional data*
> > ~~~
> > native_func_data = [b for b in func_data if b not in mni_func_data]
> > native_func_data
> > ~~~
> > {: .language-python}
> > 
> > Now that we have a handle on how fMRIPREP preprocessed data is organized and how we can pull this data. Let's start working with the actual data itself!
> {: .solution}
{: .challenge}

{% include links.md %}