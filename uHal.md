* [link](https://ipbus.web.cern.ch/doc/user/html/software/uhalQuickTutorial.html) to further explanation
* Used for connecting to [[IPbus]]
	* IPbus is an VHDL interface framework for making data and interfaces accessible through registers
* Addresses are stored in [XML](https://en.wikipedia.org/wiki/XML) files.
	* Per address, information about the registers can be given. For example:
		* id
		* mask
		* description
		* mode *(single, block, incremental, non-incremental)*
		* size
		* permission *(read, write, readwrite, r, w, rw)*
		* relative address *(only when an absolute address is given)*
		* another [xml](https://en.wikipedia.org/wiki/XML) file with addresses and information
		* fwinfo *(for slave busses only. can also determine width of bus)*
## Usage

Communicating with the IPbus is done in a number of steps.

1. Open an connection to the device using the ```ConnectionManager```.
2. Using the ```HwInterface``` class, retrieve any registers mentioned in the XML file.
3. After retrieving the registers, execute a read or write method on that registers.
	* **Note: The actual read and write actions have not been performed yet!!**
4. execute the dispatch function to trancseive the data.