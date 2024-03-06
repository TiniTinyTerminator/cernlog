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
### Example address table
``` XML 
<?xml version="1.0" encoding="ISO-8859-1"?>

<!--group-->
<node>
	<!--subgroup-->
	<node id="C1">
		<!--register with absolute address-->
		<node id="A1" address="0x00000200" permission="rw">
			<!--mask-->
			<node id="G0" mask="0x01" />
			<node id="G1" mask="0x02" />
			<node id="G2" mask="0x04" />
			<node id="G3" mask="0x08" />
			<node id="G4" mask="0xF0" />
		<node/>
		
		<node id="A2" address="0x00000201" permission="w" />
		<node id="A3" address="0x00000202" permission="r" />
		<node id="A4" address="0x00000203" permission="readwrite" />
	</node>
	
	<!--subgroup with absolute address-->
	<node id="C2" address="0x00000300">
		<!--register with relative address-->
		<node id="A1" address="0x000" />
		<node id="A2" address="0x001" />
		<node id="A3" address="0x002" />
		<node id="A4" address="0x003" />
	</node>

	<!--external address table-->
	<node id="system" module="file://external.xml" address="0x00000400" />
</node>
```
## Usage

Communicating with the IPbus is done in a number of steps.

1. Open an connection to the device using the ```ConnectionManager```.
2. Using the ```HwInterface``` class, retrieve any registers mentioned in the XML file.
3. After retrieving the registers, execute a read or write method on that registers.
	* **Note: The actual read and write actions have not been performed yet!!**
4. execute the dispatch function to trancseive the data.

### Example 1: readout of single register
```CPP
#include "uhal/uhal.hpp"

int main() {
    // Initialize the uHAL log level
    uhal::setLogLevelTo(uhal::LogLevel::NOTICE);

    // Connection settings - specify the .xml connection file and the device ID
    const std::string connectionFile = "path/to/connectionFile.xml";
    const std::string deviceId = "yourDeviceId";

    // Create a uHAL connection manager
    uhal::ConnectionManager manager("file://" + connectionFile);
    uhal::HwInterface hardware = manager.getDevice(deviceId);

    // Memory block settings - base address and size (in number of 32-bit words)
    const std::string address = "registerName";

	// Retrieve the register
	uhal::ValWord<uint32_t> reg = hardware.getNode(address).read();
	
	// Excecute the transaction
	hardware.dispatch();

	// Print the value of the register
	std::cout << reg.value() << std::endl;

	return 0;
}

```

### Example 2: readout of FIFO
```CPP
#include "uhal/uhal.hpp"

int main() {
    // Initialize the uHAL log level
    uhal::setLogLevelTo(uhal::LogLevel::NOTICE);

    // Connection settings - specify the .xml connection file and the device ID
    const std::string connectionFile = "path/to/connectionFile.xml";
    const std::string deviceId = "yourDeviceId";

    // Create a uHAL connection manager
    uhal::ConnectionManager manager("file://" + connectionFile);
    uhal::HwInterface hardware = manager.getDevice(deviceId);

    // Define the FIFO status and data registers' names as per your hardware configuration
    const std::string fifoStatusRegister = "fifoStatus";
    const std::string fifoDataRegister = "fifoData";

    // Continuously check FIFO status and read data if available
    while (true) {
        // Read the FIFO status register to determine if data is available
        // Assume the status register is set to 0 when FIFO is empty
        uhal::ValWord<uint32_t> status = hardware.getNode(fifoStatusRegister).read();
        hardware.dispatch();

        if (status.value() == 0) {
            // FIFO is empty, no more data to read
            std::cout << "FIFO is empty, no more data to read." << std::endl;
            break; // Exit the loop if the FIFO is empty
        } else {
            // FIFO has data, read the next data word
            uhal::ValWord<uint32_t> dataWord = hardware.getNode(fifoDataRegister).read();
            hardware.dispatch();

            // Process the data word
            std::cout << "Read data word: " << std::hex << dataWord.value() << std::endl;
        }

        // Implement any necessary delays or checks here if needed, to avoid overwhelming the hardware or the IPbus interface
    }

    return 0;
}
```

### Example 3: readout of memory block
```CPP
#include "uhal/uhal.hpp"

int main() {
    // Initialize the uHAL log level
    uhal::setLogLevelTo(uhal::LogLevel::NOTICE);

    // Connection settings - specify the .xml connection file and the device ID
    const std::string connectionFile = "path/to/connectionFile.xml";
    const std::string deviceId = "yourDeviceId";

    // Create a uHAL connection manager
    uhal::ConnectionManager manager("file://" + connectionFile);
    uhal::HwInterface hardware = manager.getDevice(deviceId);

    // Memory block settings - base address and size (in number of 32-bit words)
    const std::string memoryBlockBaseAddress = "memoryBlockBase";
    const uint32_t memoryBlockSize = 1024; // Example size, adjust as needed

    // Loop over the memory block and read each word
    for (uint32_t i = 0; i < memoryBlockSize; ++i) {
        // Calculate the address of the current word
        std::string currentAddress = memoryBlockBaseAddress + "+" + std::to_string(i*4); // Assuming 32-bit words, address offset by 4 bytes

        // Read the current word from the memory block
        uhal::ValWord<uint32_t> dataWord = hardware.getNode(currentAddress).read();
        hardware.dispatch();

        // Process the data word
        std::cout << "Read data word at address " << currentAddress << ": " << std::hex << dataWord.value() << std::endl;
    }

    // Complete any additional processing or cleanup here

    return 0;
}

```