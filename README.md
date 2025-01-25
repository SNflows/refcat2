# REFCAT2

CMake project to ease the setup of system-wide `refcat` executable with binary (rather than csv) data enabled.

All credit goes to [J. L. Tonry et al.](https://archive.stsci.edu/hlsp/atlas-refcat2)

Please remember to cite the appropriate paper(s) and the DOI found in the link above if you use these data in a published work.

The data and software are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

## Installation

```shell
git clone https://github.com/SNflows/refcat2.git
cd refcat2
mkdir build
cd build
cmake ..
cmake --build .
sudo cmake --install .
```

## Notes

### Data archives

When `cmake ..` is invoked, CMake looks for the files

* `hlsp_atlas-refcat2_atlas_ccd_00-m-16_multi_v1_cat.tbz`
* `hlsp_atlas-refcat2_atlas_ccd_16-m-17_multi_v1_cat.tbz`
* `hlsp_atlas-refcat2_atlas_ccd_17-m-18_multi_v1_cat.tbz`
* `hlsp_atlas-refcat2_atlas_ccd_18-m-19_multi_v1_cat.tbz`
* `hlsp_atlas-refcat2_atlas_ccd_19-m-20_multi_v1_cat.tbz`

in the directory `build`.

If CMake does not find these files, it downloads them to `build` from `https://archive.stsci.edu/hlsp/atlas-refcat2`.
This currently happens serially, not in parallel; you may want to download the files yourself...

### Extracted data

When `cmake ..` is invoked, after CMake has looked for and possibly found the archives listed above, 
CMake then checks for the existence of the directories

* `00_m_16`
* `16_m_17`
* `17_m_18`
* `18_m_19`
* `19_m_20`

in the directory `build`.

If CMake does not find these directories, it extracts the archives listed above to the corresponding directories listed here.
This currently happens serially, not in parallel; you may want to extract the archives yourself...

### Refcat binary

When `cmake --build .` is invoked, the `refcat` binary is first built, and all extracted data are then converted from csv to binary format via the extra build target

```shell
refcat 0 0 -dir <paths to extracted data directories> -CSV_to_binary <DATADIR>/refcat 
```

with output to `<DATADIR>/refcat`.

As with the downloads and extractions, this step takes a while.

Re-running `cmake --build .` repeats the csv-to-binary step.

### Installing refcat

When `sudo cmake --install .` is invoked, the `refcat` binary and the binary data files, generated from the csv files, are installed to the system's default install paths.

`refcat` can then be invoked from a terminal anywhere, and `refcat` will refer to the binary data files which are now installed system-wide.

After running `sudo cmake --install .`, it is safe to clear the `build` directory.

## Todo

* Parallelise download and extraction of data
* Proper man page installation
