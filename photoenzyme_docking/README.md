# Photoenzyme Docking
This document serves as a guide as to our docking protocol for the study of the Flavin based
photoenymes.

The overall process is as follows
1. Start with the base protein (PDB: 6myw) that has already been preprocessed
1. Dock a variant of `FMNhq` onto 6myw
1. Docking `CHL` then `STY`
    1. For each resulting pose from step 2 dock `CHL`
    1. For each resulting pose from step 3.1 dock `STY`
1. Docking `STY` then `CHL`
    1. For each resulting pose from step 2 dock `STY`
    1. For each resulting pose from step 4.1 dock `CHL`

## Step 0 - Naming Convention
Before we do any calculations we need to be organized and use the same naming convention so we all
know what calculations we are referring to. I expect you to use the same convention.

### Flavin
For this project we are particularly looking at **F**lavin **m**ono**n**ucleotide or `FMN` for short.
In particular we are looking at a fully reduced form of FMN which is named FMN **H**ydro**q**uinone
or `FMNhq` for short.

The last thing to note about FMNhq is the Phosphate group on its tail.
This phosphate group has three protonation states
1. Dihydrogen **Ph**osphate Group (`Ph` for short)
1. Hydrogen **Ph**osphate Group (`Ph-1` for short)
1. **Ph**osphate Group (`Ph-2`)

The number on the end of these abbreviations reflect the charge of phosphate groups in each of these
protonation states. So `Ph` is neutral, `Ph-1` has a charge of -1, and `Ph-2` has a
charge of -2.

### Substrates
There are two substrates that are involved in this particular system we are studying

1. N,N-dimethyl**chl**oroamide (`CHL` for short)
1. α-methyl**sty**rene (`STY` for short)

## Step 1 - Setup
You'll first need to make a new project. Launch a Maestro job here

https://ood.discovery.neu.edu/pun/sys/dashboard/batch_connect/sys/Maestro/session_contexts/new

I recommend using the gpu with 4 cpus and 8GB of memory. Once the job loads.

1. Go to `File -> New Project` you can name the project whatever so long as it helps you stay organized.
2. Then got to `Edit` and navigate to the `General -> Directories` section (this should be the default already) and make sure `Project jobs directory` is selected

I've already optimized the protein and substrates for the docking so you'll need to import these.

1. Go to `File -> Import Structures` and navigate to `/scratch/chintala.v/schrodinger/docking_start.prj/input`
2. Select all of the files in this directory and then hit `Open`

You should now have two new groups in your entry list named `protein_prep` and `substrates - M06-2X / 6-311+G(d,p)`. These groups are respectively the optimized protein and corresponding substrates.

## Step 2 - Docking FMN
Before we dock your assigned flavin structure we first need to generate a grid for the protein. 

1. To start select and include the `6MYW - minimized` structure.
2. Then go to `Tasks` in the top right and search for `Receptor Grid Generation`
3. Select one of the atoms in the Flavin in the protein (these atoms should look green)
4. Check the `Use input partial charges` checkbox
5. Name this job `6myw_gridgen` and hit run

Once the gridgen job is complete we can dock on flavin structure. You may have a different assigned
Phosphate state but for the sake of example for this tutorial I will use `FMNhq_Ph-2` just replace this
with whatever specific FMN structure you are doing i.e. `FMNhq_Ph` or `FMNhq_Ph-1`.

1. Include and select `FMNhq_Ph-2` in the substrates group
1. Go to `Tasks` and search for `Ligand Docking`
1. Make sure `Receptor grid` is set to `From file` then hit `Browse` and search for
your grid file it should be in a directory labeled `6myw_gridgen` in your `jobs` folder where your project is located.
1. Then in the `Ligands` tab make sure `Use ligands from` is set to `Workspace`
1. Check the `Use input partial charges box`
1. Next go to the `Settings` tab and set the `Precision` drop down to `XP` (extra precision)
1. Check the `Write XP descriptor information` checkbox
1. Next go to the `Output` tab and set the `Write out at most:` to 10
1. Name this job `FMNhq_Ph-2_glide`
1. Hit the smaller gear icon at the bottom right and click `Job Settings`
1. Make sure your host is `discovery-general (48)` and the `CPUs` textbox is set to 48

Once that job completes it should auto incorporate the poses from the docking. If this doesn't happen there could have been something else that happened contact me.
The new group should be called `FMNHq_Ph-2_glide_pv`. The first entry in this group
should be `6myw - minimized` followed by potentially more than one entry all named `FMNhq_Ph-2` these represent each of the docking poses.

You can view these poses if you are curious. The next step is to merge these structures.

1. First select the `6myw - minimized` in this new group.
1. Then multi-select (hold Ctrl to do this) the *first* entry of `FMNhq_Ph-2` in this new group.
1. Right click and then merge these structures. Name this merged structure `FMNhq_Ph-2_1`
1. Merge the rest of the docking poses following the same convention. So merge `6myw - minimized` and the next `FMNhq_Ph-2` entry and rename it `FMNhq_Ph-2_2` etc.

Now that we have the merged structures we need to generate a new grid for each of these molecules. Because we are doing multi-substrate docking this process is slightly different now. So for each of the merged structures from the last step do the following steps, for example I will use `FMNHq_Ph-2_1`

1. Navigate to `Receptor Grid Generation`
1. *Uncheck* `Pick to identify ligand` and `Show markers`
1. Check `Use input partial charges`
1. Navigate to the `Site` tab
1. Set the X, Y, and Z coordinates to `15.52 5.32 53.61` respectively
1. On the `Size` subsection of the `Site` tab set the `Dock ligand with length` to 14
1. Hit run. You *should* get a popup box saying "The grid site contains one...", this is expected and what we want. If you don't see this popup then something has gone wrong. Hit `Continue` on this popup.

## Step 3 - Docking the First Substrate
So there are two orders of docking the substrates `CHL` then `STY` or `STY` then `CHL` we are going to do both. For this tutorial I'll just go through docking `CHL` then `STY`.

Now that we have poses of the protein and our FMN structure we can dock the next structure, again for this example this will be CHL.

You should do this with each of the generated `FMNhq_Ph-2` gridgen poses we have from the last step.

Essentially all the steps for docking are the same as Step 2. The only things that's changed is where our grid file is and what substrate we dock. Use the `CHL` and `STY` structure that we imported in the `substrates` group. When you run the job make sure you are using 48 CPUs. Lastly we giving a name for these jobs make note of what pose you are using and what substrates you are docking. So for this example I would call my job `FMNhq_Ph-2_1_CHL_glide`

Once the docking job is complete we need to merge the structures again just as before. Again same steps all that has changed is we have new names. So for my first merge name is `FMNhq_Ph-2_1_CHL1`. Note I don't put another underscore before the number here I only did that from FMNhq because it ends in a number. The next merge is `FMNhq_Ph-2_1_CHL2` etc.

Again like last time we need to generate a grid for each of our merged structures. Use the same steps outline as the last part of Step 2 where we explicitly enter in the XYZ center.

## Step 4 - Docking the Second Substrate
We essentially repeat all of Step 3 but now with our next substrate. No carrying on the example I would use a grid for `FMNhq_Ph-2_1_CHL1` and dock STY with a job name `FMNhq_Ph-2_1_CHL1_STY`.

Once the docking is done again merge the structure and name them with the number system. So the first merge would be `FMNhq_Ph-2_1_CHL1_STY1`

Especially when docking `STY` then `CHL` you may run into scenarios where this second docking doesn't produce any poses that's fine. If you run into this let me know and I will double check.

## Step 5 - Exporting
If you're reading to this point you may be thinking this is a bit of exponential growth of calculations and you would be correct. Suppose our FMN docking produced 3 poses and when docking the first substrate CHL and STY both produce 3 poses. And when docking the second substrates STY and CHL produce 3 poses we'd have 2 * (3 * 3 * 3) = 54 poses with the following names
```
FMNHq_Ph-2_1_CHL1_STY1
FMNHq_Ph-2_1_CHL1_STY2
FMNHq_Ph-2_1_CHL1_STY3
FMNHq_Ph-2_1_CHL2_STY1
FMNHq_Ph-2_1_CHL2_STY2
FMNHq_Ph-2_1_CHL2_STY3
FMNHq_Ph-2_1_CHL3_STY1
FMNHq_Ph-2_1_CHL3_STY2
FMNHq_Ph-2_1_CHL3_STY3
FMNHq_Ph-2_2_CHL1_STY1
FMNHq_Ph-2_2_CHL1_STY2
FMNHq_Ph-2_2_CHL1_STY3
FMNHq_Ph-2_2_CHL2_STY1
FMNHq_Ph-2_2_CHL2_STY2
FMNHq_Ph-2_2_CHL2_STY3
FMNHq_Ph-2_2_CHL3_STY1
FMNHq_Ph-2_2_CHL3_STY2
FMNHq_Ph-2_2_CHL3_STY3
FMNHq_Ph-2_3_CHL1_STY1
FMNHq_Ph-2_3_CHL1_STY2
FMNHq_Ph-2_3_CHL1_STY3
FMNHq_Ph-2_3_CHL2_STY1
FMNHq_Ph-2_3_CHL2_STY2
FMNHq_Ph-2_3_CHL2_STY3
FMNHq_Ph-2_3_CHL3_STY1
FMNHq_Ph-2_3_CHL3_STY2
FMNHq_Ph-2_3_CHL3_STY3
FMNHq_Ph-2_1_STY1_CHL1
FMNHq_Ph-2_1_STY1_CHL2
FMNHq_Ph-2_1_STY1_CHL3
FMNHq_Ph-2_1_STY2_CHL1
FMNHq_Ph-2_1_STY2_CHL2
FMNHq_Ph-2_1_STY2_CHL3
FMNHq_Ph-2_1_STY3_CHL1
FMNHq_Ph-2_1_STY3_CHL2
FMNHq_Ph-2_1_STY3_CHL3
FMNHq_Ph-2_2_STY1_CHL1
FMNHq_Ph-2_2_STY1_CHL2
FMNHq_Ph-2_2_STY1_CHL3
FMNHq_Ph-2_2_STY2_CHL1
FMNHq_Ph-2_2_STY2_CHL2
FMNHq_Ph-2_2_STY2_CHL3
FMNHq_Ph-2_2_STY3_CHL1
FMNHq_Ph-2_2_STY3_CHL2
FMNHq_Ph-2_2_STY3_CHL3
FMNHq_Ph-2_3_STY1_CHL1
FMNHq_Ph-2_3_STY1_CHL2
FMNHq_Ph-2_3_STY1_CHL3
FMNHq_Ph-2_3_STY2_CHL1
FMNHq_Ph-2_3_STY2_CHL2
FMNHq_Ph-2_3_STY2_CHL3
FMNHq_Ph-2_3_STY3_CHL1
FMNHq_Ph-2_3_STY3_CHL2
FMNHq_Ph-2_3_STY3_CHL3
```

In practice we won't get this many I'd say maybe around 30 at most. Select all of these structures and
1. Go to `File -> Export Structures`
1. For `Files of Type` choose `PDB (*.pdb *.pdb.gz *.ent *.ent.gz)`
1. Expand the `Options` section if it is hidden.
5. Make sure that `Structure source to be exported` is set to `Project Table (selected entries)`
6. Choose `Files: Export each entry individually`
8. Click `Save` and put these in a location you know and let me know.
