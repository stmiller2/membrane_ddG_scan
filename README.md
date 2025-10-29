# Membrane Protein ddG Scan Pipeline - Version 2
**Updated:** 02/14/2025  
**Contact:** stmiller2@wisc.edu

---

## Overview
This pipeline computes Rosetta ΔΔG scores for all possible single missense mutations of a membrane protein. It takes inspiration from Tony Meger's soluble protein ddG pipeline and the Tiemann 2023 Biophysical Journal paper. More notes on modifications can be found in the code comments.

This pipeline was developed for the Biochemistry Compute Cluster but can be adapted to any system with HTCondor.

---

## Usage Instructions

### Prepare Files

**Step 1:** Clone the ddG pipeline GitHub repo and rename it to reflect your experiment.
```bash
git clone https://github.com/stmiller2/membrane_ddG_scan
mv membrane_ddG_scan/ my_experiment/
```

**Step 2:** Orient your protein in the membrane using the OPM database:  
- [OPM](https://opm.phar.umich.edu/)  
- [PPM 3.0](https://opm.phar.umich.edu/ppm_server3_cgopm)  

Instructions:  
- Search for your protein in the database or upload a PDB file to the PPM 3.0 webserver.  
- Multiple chains are okay; only extract the chain of interest later.  
- Download the transformed file and verify the membrane orientation in PyMol.  
- Copy the transformed PDB (`prot_tr.pdb`) to the `inputs` directory of the ddG pipeline.

**Step 3:** Copy the directory to BCC:  
```bash
cp -r ddG_v2 /scratch/{username}/
```

**Step 4:** Open `prep_inputs.sh` in a text editor and set the user variables.

**Step 5:** Run `prep_inputs.sh` to clean the PDB, generate a spanfile, and relax the protein with Rosetta:

```bash
chmod +x prep_inputs.sh
./prep_inputs.sh
```
- Double-check the generated spanfile (`prot_tr_A.span`) for correct transmembrane domains. Note that Rosetta numbering may differ from your original PDB.  
- Relaxation may take ~30 minutes depending on protein size.  
- Optionally, compare the relaxed PDB (`prot_tr_A_0001.pdb`) with the original in PyMol to confirm minimal backbone movement.

---

### Run ddG Scan

**Step 6:** Open `mp_cartddG_pipeline.sh` in a text editor and set the user variables.  
- Start with `FULL_RUN=false` to test with M1A, then switch to `FULL_RUN=true` for the full scan.

**Step 7:** Run `mp_cartddG_pipeline.sh` to generate directories and mutfiles, create bash scripts for `cartesian_ddg`, and submit HTCondor jobs.
```bash
chmod +x mp_cartddG_pipeline.sh
./mp_cartddG_pipeline.sh
```  
- Duration depends on protein size.  
- Monitor jobs with `condor_q` or `condor_tail -f {jobid}`.

---

### Parse and Save Results

**Step 8:** Open `parse_results.sh` in a text editor and set the user variables.

**Step 9:** Run `parse_results.sh` to compute ΔΔG values.
```bash
chmod +x parse_results.sh
./parse_results.sh
```
- This loops through ddG output files, averages WT and MUT total energies, and computes ΔΔG.  
- Rosetta numbering is used; adjust for missing residues in the original PDB.

**Step 10:** Save data and clean up. Recommended:  
```bash
tar -czf ddG_scan.tar.gz ddG_v2/
```
- Move the tar file to a fileserver and clear the scratch directory.

**Step 11:** Publish CNS
