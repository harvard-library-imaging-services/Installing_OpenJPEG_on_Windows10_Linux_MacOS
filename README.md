## Installing OpenJPEG on Windows 10

[OpenJPEG](https://www.openjpeg.org/) is an open-source JPEG 2000 codec written in C. There is no GUI. This command-line tool is easiest to run and maintain via a command-line friendly OS like Linux, so I recommend installing Linux on your Windows 10 machine, taking advantage of W10's [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) feature.

### 1. Install Ubuntu 20.04 LTS

* From the Windows App Store, **download** [Ubuntu](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71?cid=msft_web_chart&activetab=pivot:overviewtab).

* Follow [Microsoft's installation instructions](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

* Update the installation by running the following command: `sudo apt-get update ; sudo apt-get upgrade -y`

* Install some additional software modules: `sudo apt-get install jasper exiftool git git-lfs cmake liblcms2-dev libtiff-dev libpng-dev libz-dev unzip`.

* Install OpenJPEG by compiling the source code:

	* Download the source code: `wget https://github.com/uclouvain/openjpeg/archive/master.zip`
		* Unzip: `unzip master.zip`
		* `cd openjpeg-master/`
		* Follow the installation instructions, per *[OpenJPEG installation](https://github.com/uclouvain/openjpeg/blob/master/INSTALL.md#openjpeg-installation)*.


> sudo ldconfig
> 

* Install imagemagick from source code. **Download**: `wget https://github.com/ImageMagick/ImageMagick/archive/master.zip`
	* `unzip master.zip`
	* Follow the [ImageMagick installation instructions](https://github.com/ImageMagick/ImageMagick/blob/master/Install-unix.txt) 

---

* Install some additional modules: ` sudo apt-get install graphicsmagick imagemagick`

### Test your installation

* [Download a test image](https://github.com/harvard-library-imaging-services/images/raw/master/001.tif)

* Run the following commands:

  `convert 001.tif 001_via_imagemagick.jp2` # test imagemagick installation

  `opj_compress -i 001.tif -o 001.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -I -q 42` # test openJPEG installation
  
### Encoding

* Command: `opj_compress`
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

`opj_compress -i D9_NS.tif -o D9_NS.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2`

### Lossy example

* `opj_compress -i D9_NS.tif -o D9_NS.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -I -q 42`
* `opj_compress -i D9_NS.tif -o D9_NS.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -r 10`

### [_decompress_](http://manpages.ubuntu.com/manpages/cosmic/man1/opj_decompress.1.html): converting JP2 files to other formats

`opj_decompress -i infile.j2k -o outfile.png`

`opj_decompress -ImgDir images/ -OutFor tif`

  
  ---
  
  end
  
  
