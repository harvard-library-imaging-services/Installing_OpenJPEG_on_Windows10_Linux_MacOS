
# Installing OpenJPEG on Windows 1X, Linux, and MacOS

[OpenJPEG](https://www.openjpeg.org/) is an open-source JPEG 2000 codec written in C. There is no GUI. This command-line tool is easy to run and maintain using a command-line friendly OS like Linux, so I recommend installing Linux on your Windows 10 machine, taking advantage of W10's [**Windows Subsystem for Linux**](https://docs.microsoft.com/en-us/windows/wsl/) feature.

Below are instructions for compiling **OpenJPEG**, **ImageMagick**, and **GrokImageCompression** from source code.

While it is easier to install OpenJPEG and ImageMagick via pre-compiled versions using **[apt](https://en.wikipedia.org/wiki/APT_(software))**, I've had issues with prior installations that I resolved by compiling the source code, so that is what I recommend. Note that you do not need to install ImageMagick at all, but it is a very useful utility.

> If you'd like to try installing OpenJPEG via the precompiled version first, the following command should do the trick:
>
> `sudo apt install libopenjp2-7 libopenjp2-tools`

### Why install GrokImageCompression?

The original JPEG 2000 standard only accommodated three color encoding declarations: sGray, sYCC, and sRGB. As color management gained popularity and a wider variety of [ICC](https://en.wikipedia.org/wiki/ICC_profile) display profiles were being embedded in images, the standard was amended in 2004 to accommodate the use of restricted ICC profiles. Those changes are incorporated into [ISO/IEC 15444-1:2016
Information technology — JPEG 2000 image coding system: Core coding system — Part 1](https://www.iso.org/standard/70018.html), the 2016 update to the standard.

As of the *openjp2 library v2.3.1.*, OpenJPEG does not carry over the ICC display profiles embedded within the source image, but instead converts any color encoding to sRGB on output.

> * [Example showing how to perform an ICC profile-to-profile conversion using ImageMagick](https://gist.github.com/comstock/73b4d456f725e3636b79da38dbb6d9df#example-showing-how-to-perform-an-icc-profile-to-profile-conversion-using-imagemagick).

[GrokImageCompression](https://github.com/GrokImageCompression/grok) is an open-source JP2 encoder based on the OpenJPEG code that produces JP2 images encoded with the same colorspace -- includes the same ICC display profile -- resident in the source image.

#### Want to run Grok Image Compression on a Mac?

It can be installed via [Homebrew](https://brew.sh/) with the command: `brew install grokj2k`

## 1. Install Ubuntu 20.04 LTS

> From the Windows App Store, **download** [Ubuntu](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71?cid=msft_web_chart&activetab=pivot:overviewtab).

> Follow [Microsoft's installation instructions](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

* Update the installation by running the following command: `sudo apt-get update ; sudo apt-get upgrade -y`

* Install some additional software modules: `sudo apt-get install pkg-config valgrind emacs-nox libltdl-dev libtiff-tools exiftool git git-lfs cmake liblcms2-dev libtiff-dev libpng-dev libheif1 libheif-dev libz-dev unzip libzstd-dev libwebp-dev build-essential hwinfo ; sudo python get-pip.py ; sudo apt-get install python3-jpylyzer`.

## 2. Install OpenJPEG by compiling the source code:
* `mkdir ~/install`
* `cd ~/install`
* Download the source code: `wget https://github.com/uclouvain/openjpeg/archive/master.zip`

> Follow the installation instructions, per [OpenJPEG installation](https://github.com/uclouvain/openjpeg/blob/master/INSTALL.md#openjpeg-installation).

* `unzip master.zip`
* `cd openjpeg-master/`
* `mkdir build`
* `cd build`
* `cmake -DCMAKE_BUILD_TYPE=Release ..`
* `make`
* `sudo make install`
* `sudo make clean`

## 3. Install ImageMagick from source code:

> Follow the [ImageMagick installation instructions](https://github.com/ImageMagick/ImageMagick/blob/master/Install-unix.txt)

* `cd ~/install`
* `rm master.zip`
* `wget https://github.com/ImageMagick/ImageMagick/archive/refs/heads/main.zip`
* `unzip main.zip`
* `cd ImageMagick-main`

> Make sure system sees the libopenjp2.pc file just created by `config`. To look for the libopenjp2.pc files: `find ~ -type f | grep libopenjp2.pc`

* `sudo pkg-config PKG_CONFIG_PATH=~/install/openjpeg-master/build`
* `./configure --enable-shared`
* `sudo make`
* `sudo make install`
* `sudo make clean`

## 4. Install GrokImageCompression

The process of compiling *Grok* is so fussy that, as of March 2023, we recommend:

1. [Install Homebrew](https://brew.sh/) on your Linux instance
2. Install [grokj2k](https://formulae.brew.sh/formula/grokj2k#default) with: `brew install grokj2k`


## 5. Test your installation

* `mkdir ~/images`
* `cd ~/images`
* `wget -O in.tif https://storage.googleapis.com/linked_content/icc/ICC_profile_test_image_Farbkreis_120grad.tif`

### Run the following commands:

* `sudo ldconfig`

> Testing ImageMagick installation

* `convert in.tif out_via_imagemagick.jp2 | exiftool out_via_imagemagick.jp2 | less`

> Testing OpenJPEG installation (lossy compression example)

* `opj_compress -i in.tif -o out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -I -q 42`

> Testing GrokImageCompression installation (lossy compression example)

* `grk_compress -i in.tif -o grk_out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -I -q 42`

> Are the JP2 files you created standard-compliant?

* `jpylyzer *.jp2 | grep '<isValid format="jp2">True</isValid>'`

---

### Encoding JP2 files

* Command: `opj_compress` and `grk_compress`
* Switches:
	* `-i <inputFile.ext>`
	*  `-p RLCP` (progression order)
	*  `-t 1024,1024` (tile size)
	*  `-EPH` (End of Packet Header marker)
	*  `-SOP` (Start of Packet)
	*  `-ImgDir <image dir>` (cannot be used with **-i** switch)
	*  `-OutFor <ext>` (extension for output files)
	*  `-o <outputFile.ext>`
	*  `-I` (specifies lossy encoding)
	*  `-q <quality in db>`
	*  `-r <compression ratio>`
	*  `-OutFor [J2K|J2C|JP2]`

    > Output format used to compress the images read from the directory specified with`-ImgDir`. Required when`-ImgDir`option is used. Supported formats are`J2K`,`J2C`, and`JP2`.
    * `-InFor [pbm|pgm|ppm|pnm|pam|pgx|png|bmp|tif|raw|rawl|jpg]`

     > Input format. Will override file tag.

### Selected Grok-specific switches
(See: [Options](https://github.com/GrokImageCompression/grok/wiki/3.-grk_compress#options) for complete list)

* `-logfile [output file name]`

> Log to file. File name will be set to`output file name`

* `-num_threads [number of threads]`

> Number of threads used for T1 compression. Default is total number of logical cores.

* `-Comment [comment]`

> Add `<comment>` in comment marker segment(s). Multiple comments (up to a total of 256) can be specified, separated by the `|` character. For example: `-C "This is my first comment|This is my second` will store `This is my first comment` in the first comment marker segment, and `This is my second` in a second comment marker.

* `-Q, -CaptureRes [capture resolution X,capture resolution Y]`

> Capture resolution in pixels/metre, in double precision.

> * If the input image has a resolution stored in its header, then this resolution will be set as the capture resolution, by default.
> * If the`-Q` command line parameter is set, then it will override the resolution stored in the input image, if present.
> * The special values `[0,0]` for `-Q` will force the encoder to **not** store capture resolution, even if present in input image.

### Lossless example

* `opj_compress -i in.tif -o out_lossless.jp2 -p RLCP -t 1024,1024 -EPH -SOP`

* `grk_compress -i in.tif -o grk_out_lossless.jp2 -p RLCP -t 1024,1024 -EPH -SOP`

### Lossy example

* `opj_compress -i in.tif -o out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -I -q 42`
* `opj_compress -i in.tif -o out_lossy_r10.jp2 -p RLCP -t 1024,1024 -EPH -SOP -r 10`

---

* `grk_compress -i in.tif -o grk_out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -I -q 42`
* `grk_compress -i in.tif -o grk_out_lossy_r10.jp2 -p RLCP -t 1024,1024 -EPH -SOP -r 10`

### [Decompress](https://man.archlinux.org/man/extra/openjpeg2/opj_decompress.1.en): converting JP2 files to other formats

* `opj_decompress -i infile.j2k -o outfile.png`

* `opj_decompress -ImgDir images/ -OutFor tif`

---

* `grk_decompress -i infile.j2k -o outfile.png`

* `grk_decompress -ImgDir images/ -OutFor tif`

—

### Encoding entire directories

* `opj_compress -OutFor jp2 -ImgDir /images/in  -p RLCP -t 1024,1024 -EPH -SOP -OutDir /images/out`


* `grk_compress -out_fmt jp2 -in_dir /images/in -p RLCP -t 1024,1024 -EPH -SOP -out_dir /images/out`

### Shell scripts

#### Lossless compression

```
#!/bin/bash
# Grok Image Compression JP2 JPEG2000 codec encoder lossless compression
# https://github.com/GrokImageCompression/grok
find . -type f | grep --extended-regexp ".*\.tif$" | sed -E "s/.tif//g" > tiffNameStem.txt
ls -1 *.tif | wc -l | xargs echo "TIFF count: "
while read line
do
#    grk_compress -i $line".tif" -o $line".jp2" -p RLCP -t 1024,1024 -EPH -SOP
    grk_compress -i $line".tif" -o $line"_lossless.jp2" -p RLCP -t 1024,1024 -EPH -SOP
    echo "converting "$line".tif"
done < tiffNameStem.txt
echo "
     ~~~ FIN ~~~
"
ls -1 *.jp2 | wc -l | xargs echo "JP2 count: "

```

#### Lossy compression

```
#!/bin/bash
#Grok Image Compression JP2 JPEG2000 codec encoder lossy compression
#https://github.com/GrokImageCompression/grok
find . -type f | grep --extended-regexp ".*\.tif$" | sed -E "s/.tif//g" > tiffNameStem.txt
ls -1 *.tif | wc -l | xargs echo "TIFF count: "
while read line
do
#grk_compress -i $line".tif" -o $line".jp2" -p RLCP -t 1024,1024 -EPH -SOP -I -q 42
    grk_compress -i $line".tif" -o $line"_lossy.jp2" -p RLCP -t 1024,1024 -EPH -SOP -I -q 42
    echo "converting "$line".tif"
done < tiffNameStem.txt
echo "
     ~~~ FIN ~~~
"
ls -1 *.jp2 | wc -l | xargs echo "JP2 count: "
```

---

  end
