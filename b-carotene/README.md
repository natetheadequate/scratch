# Gaussian: TD-DFT Calculations
We are going to be running some calculations on β-Carotene. This organic is what gives
orange vegetables / fruits (pumpkins, mangoes, carrots, sweet potatoes..) their distinct
orange color. We'll use TD-DFT to show why this compound reflects orange light.

https://en.wikipedia.org/wiki/%CE%92-Carotene

## Initial Setup
To stay organized I would make a directory in your `/work` directory called gaussian. Then
within `/work/$USER/gaussian` make another directory called b-carotene. Once those are made
you can `cp` all these files into that directory and `cd` into it. The below commands will
do this.

```bash
mkdir -p /work/$USER/gaussian/b-carotene
cp /work/donglab/chintala.v/reu/gaussian/* /work/$USER/gaussian/b-carotene
cd /work/$USER/gaussian/b-carotene
```

## Downloading an Initial Structure
To practice using the command line we are going to download this structure from the web.
Copy paste the below command into the terminal and hit enter.

```bash
wget -O b-carotene.sdf 'https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/CID/5280489/record/SDF/?record_type=3d&response_type=save&response_basename=Conformer3D_CID_5280489'
```

This will download a `sdf` file which is a coordinate file format for the structure of β-Carotene.

## Viewing β-Carotene in GaussView
Now that we have a structure for β-Carotene we can view it in GaussView. Got to the below link
and launch a GaussView job on OOD. You can use a GPU if you would like.

https://ood.discovery.neu.edu/pun/sys/dashboard/batch_connect/sys/Gaussian_gview/session_contexts/new

Go to `File -> Open` and browse to your saved `sdf` file and open it. You should now be able to see
β-Carotene, the bottom left should say `96 atoms, 296 electrons, neutral, singlet`.

## Setting Up Calculations
Now back to the terminal in the same directory make two new directories called `opt` and `tddft`.

No go back to GaussView and `File -> Save`
1. Change the `Files of type` to `Gaussian Input File (*.gjf, *.com)`
2. Save this file into the `opt` directory

## Editing Gaussian Job File
Open the `gjf` file in the `opt` directory in whatever text editor you like to use. The first four
lines should be

```
%chk=b-carotene.chk
# hf/3-21g geom=connectivity

5280489
```

Edit these lines so that the top of you file looks like this
```
%nprocshared=48
%mem=120GB
%chk=b-carotene.chk
# opt 6-31+g(d) m062x

Beta-Carotene optimization

0 1
```

The next lines after the `0 1` should be atomic symbols with their coordinates

## Running The Calculations
Now while you are in `/work/$USER/gaussian/b-carotene` run the below command

```bash
sbatch submit.sh opt/b-carotene.gjf
```

This will submit the optimization job to the Slurm queue. You can check the status
of your job by running `squeue -u $USER`. You should see a job running called `g16`
with the Status `R` which stands for Running after some time.

This calculation should take around **45 minutes**

## Verifying Optimization
Gaussian always log a line that starts with `Normal Termination` after its completes a
calculations. If we see this in our log file we'll know that everything went fine.

To do this we can run `grep Normal opt/b-carotene.log` we should see one line that say
Normal.

We can also view the optimization in GaussView. Go back to GaussView and open the
`opt/b-carotene.log` file and make to sure the check the `Read Intermediate Geometries`
checkbox. Now you can step through each step of the optimization and see exactly how the
structure changed. Luckily our structure was already pretty optimized so we only needed ~15 steps.

We can also see a graph of the energy of the structure at each optimization step.
Right click on the blue background window with and go to
`Results -> Optimization`

## TD-DFT
While still in GaussView make sure you have the last optimization step selected and go to
`File -> Save`. Again save a `Gaussian Input File` but this time in the `tddft` directory.

Again open the `tddft/b-carotene.gjf` file in your text editor and this time change this line

```
# opt 6-31+g(d) m062x
```

to

```
# m062x/6-31+g(d) TD(Nstates=50)
```

Next from the `/work/$USER/gaussian/b-carotene` directory run the following command

```
sbatch submit.sh tddft/b-carotene.gjf
```

This calculation will take around **1 hours 30 minutes**

## Simulated UV-Vis Spectrum
Once the TD-DFT calculation has completed view the `tddft/b-carotene.log` file in GaussView.
On the blue background window right click `Results -> UV-Vis` this will now show the simulated
spectrum.

You should see a large peak ~430nm and smaller peaks from the 200-300nm range. The overall band
shows absorptions nears the 200-550nm range. If we look at at a visible light spectrum [here](https://en.wikipedia.org/wiki/Visible_spectrum#/media/File:Linear_visible_spectrum.svg)
we see that β-Carotene absorbs light in the UV - Green/Yellow range. Leaving the Orange/Red light to reflect. If you go back to the
wikipedia article and see what the actual compound looks like it indeed looks Orange/Red!
