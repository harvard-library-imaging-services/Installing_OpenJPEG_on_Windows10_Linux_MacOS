## Installing OpenJPEG on Windows 10

[OpenJPEG](https://www.openjpeg.org/) is an open-source JPEG 2000 codec written in C. There is no GUI. This command-line tool is easiest to run and maintain via a command-line friendly OS like Linux, so I recommend installing Linux on your Windows 10 machine, taking advantage of W10's [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) feature.

### 1. Install Ubuntu 20.04 LTS

* From the Windows App Store, **download** [Ubuntu](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71?cid=msft_web_chart&activetab=pivot:overviewtab).

* Follow [Microsoft's installation instructions](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

* Update the installation by running the following command: `sudo apt-get update ; sudo apt-get upgrade -y`

* Install some additional software modules: `sudo apt-get install exiftool git cmake liblcms2-dev libtiff-dev libpng-dev libz-dev`.

* Install OpenJPEG by compiling the source code:

	* Download the source code: `wget https://github.com/uclouvain/openjpeg/archive/master.zip`
		* Unzip: `unzip master.zip`
		* `cd openjpeg-master/`
		* Follow the installation instructions, per *[OpenJPEG installation](https://github.com/uclouvain/openjpeg/blob/master/INSTALL.md#openjpeg-installation)*.

* Install some additional modules: ` sudo apt-get install graphicsmagick imagemagick`

### Test your installation

* [Download a test image](https://github.com/harvard-library-imaging-services/images/raw/master/001.tif)

* Run the following commands:

  `convert 001.tif 001_via_imagemagick.jp2` # test imagemagick installation

  `opj_compress -i 001.tif -o 001.jp2 -p RLCP -t 1024,1024 -EPH -SOP -OutFor jp2 -I -q 42` # test openJPEG installation
  
### Encoding

<script src="https://gist.github.com/comstock/b8abed5b0e75a4b2f531f0489231ccbb.js"></script>
  
  ---
  
  end
  
  
