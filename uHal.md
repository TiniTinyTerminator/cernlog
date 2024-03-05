* [link](https://ipbus.web.cern.ch/doc/user/html/software/uhalQuickTutorial.html) to further explanation
* Used for connecting to [IPbus](https://ipbus.web.cern.ch/doc/user/html/index.html) firmware
	* IPbus is an VHDL interface framework for making data accessible through registers
* Addresses are stored in xml files.
	* Per address, information about the registers can be given. for example:
		* id
		* mask
		* description
		* mode *(single, block, incremental, non-incremental)
		* size
		* permission *(read, write, readwrite, r, w, rw)
		* relative address *(only when an absolute address is given)
		* another xml file with addresses and information