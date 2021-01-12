# Installing OpenJPEG on Windows 10

[OpenJPEG](https://www.openjpeg.org/) is an open-source JPEG 2000 codec written in C. There is no GUI. This command-line tool is easiest to run and maintain via a command-line friendly OS like Linux, so I recommend installing Linux on your Windows 10 machine, taking advantage of W10's [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) feature.

Below are instructions for compiling OpenJPEG, ImageMagick, and GrokImageCompression from source code.

It is easier to install OpenJPEG and ImageMagick via pre-compiled versions using **apt**, but I've had issues with prior installations that I resolved by compiling the source code, so that is what I recommend. And you don't need to install ImageMagick at all, but it is a very useful utility.

If you'd like to try installing OpenJPEG via the precompiled version first, the following command should do the trick:

`sudo apt install libopenjp2-7 libopenjp2-tools`

## Why install GrokImageCompression?

The original JPEG 2000 standard only accommodated and color encoding declarations: sGray, sYCC, and sRGB. As color management gained users and a wider variety of color encoding profiles were used and embedded in files as display profiles, the standard was amended ([15444-1annexi.pdf](https://wiki.opf-labs.org/download/attachments/11337762/15444-1annexi.pdf?version=1&modificationDate=1324478641000)) so that embedding ICC profiles would be compliant with the standard.

As of the openjp2 library v2.3.1., OpenJPEG does not carry over the display profile embedded within the source image. OpenJPEG does convert the colorspace of input files to produce an sRGB JP2 output file.

[GrokImageCompression](https://grokimagecompression.github.io/grok/) is an open-source JP2 encoder based on the OpenJPEG code that successfully generates JP2 images encoded with the same colorspace (and ICC display profile) found in the source image.

## 1. Install Ubuntu 20.04 LTS

> From the Windows App Store, **download** [Ubuntu](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71?cid=msft_web_chart&activetab=pivot:overviewtab).

> Follow [Microsoft's installation instructions](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

* Update the installation by running the following command: `sudo apt-get update ; sudo apt-get upgrade -y`

* Install some additional software modules: `sudo apt-get install pkg-config emacs-nox python3-jpylyzer libltdl-dev libopenjp2-7-dev libtiff-tools exiftool git git-lfs cmake liblcms2-dev libtiff-dev libpng-dev libz-dev unzip libzstd-dev libwebp-dev build-essential g++`.

## 2. Install OpenJPEG by compiling the source code:
* `mkdir ~/install`
* `cd ~/install`
* Download the source code: `wget https://github.com/uclouvain/openjpeg/archive/master.zip`

> Follow the installation instructions, per [OpenJPEG installation](https://github.com/uclouvain/openjpeg/blob/master/INSTALL.md#openjpeg-installation).

* `unzip master.zip`
* `cd openjpeg-master/`
* `mkdir build`
* `cd build`
* `cmake .. -DCMAKE_BUILD_TYPE=Release`
* `make`
* `sudo make install`
* `sudo make clean`

## 3. Install imagemagick from source code:

> Follow the [ImageMagick installation instructions](https://github.com/ImageMagick/ImageMagick/blob/master/Install-unix.txt)

* `cd ~/install`
* `rm master.zip`
* `wget https://github.com/ImageMagick/ImageMagick/archive/master.zip`
* `unzip master.zip`
* `cd ImageMagick-master`

> Make sure system sees the libopenjp2.pc file just created by `config`. To look for the libopenjp2.pc files: `find ~ -type f | grep libopenjp2.pc`

* `sudo pkg-config PKG_CONFIG_PATH=~/install/openjpeg-master/build`
* `./configure --enable-shared`
* `sudo make`
* `sudo make install`
* `sudo make clean`

## 4. Install GrokImageCompression

> Follow the [GrokImageCompression installation instructions](https://github.com/GrokImageCompression/grok/blob/master/INSTALL.md)

* `cd ~/install`
* `rm master.zip`
* `wget https://github.com/GrokImageCompression/grok/archive/master.zip`
* `unzip master.zip`
* `cd grok-master`
* `mkdir build`
* `cd build`
* `cmake -DCMAKE_BUILD_TYPE=Release ..`
* `make`
* `sudo make install`
* `sudo make clean`

## 5. Test your installation

* `mkdir ~/images`
* `cd ~/images`
* `wget https://github.com/harvard-library-imaging-services/images/raw/master/in.tif`

### Run the following commands:

> Testing ImageMagick installation

* `convert in.tif out_via_imagemagick.jp2 | exiftool out_via_imagemagick.jp2 | less`

> Testing OpenJPEG installation (lossy compression example)

* `opj_compress -i in.tif -o out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -I -q 42`

> Testing GrokImageCompression installation (lossy compression example)

* `grk_compress -i in.tif -o grk_out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -I -q 42`

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
	*  `-q <quality in dB>,<quality in dB>,...`
	*  `-r <compression ratio>,<compression ratio>,...`

### Lossless example

`opj_compress -i in.tif -o out_lossless.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2`

`grk_compress -i in.tif -o grk_out_lossless.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2`

### Lossy example

* `opj_compress -i in.tif -o out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -I -q 42`
* `opj_compress -i in.tif -o out_lossy_r10.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -r 10`

---

* `grk_compress -i in.tif -o grk_out_lossy_42db.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -I -q 42`
* `grk_compress -i in.tif -o grk_out_lossy_r10.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -r 10`

### [Decompress](http://manpages.ubuntu.com/manpages/cosmic/man1/opj_decompress.1.html): converting JP2 files to other formats

* `opj_decompress -i infile.j2k -o outfile.png`

* `opj_decompress -ImgDir images/ -OutFor tif`

---

* `grk_decompress -i infile.j2k -o outfile.png`

* `grk_decompress -ImgDir images/ -OutFor tif`
  
  
---
  
  end
  
  
