# infra-ecosphere
This project implements a simulated IPMI Server and manipulate Oracle VirtualBox VM. The main purpose of this project is to solve that there is not enough server during server provisioning service development, so it maps the relationship between BMC chip (via IPMI) and Virtual Machines, so that the developer can do their test with minimal environment (One machine) if they use IPMI protocol as the tool to manipulate servers in their provisioning service implementation.
 
Note: This project supports only IPMI v1.5 currently.

![image](https://raw.githubusercontent.com/rmxymh/sandbox/master/documents/infra-ecosphere/screenshot.png)


## Current Supported Operations

* Operations
    * Chassis Power On
    * Chassis Power Off
    * Chassis Power Reset / Cycle
    * Chassis Power Soft
    * Chassis Set system boot device (PXE, Disk, local CD/DVD)
* App Authentication
* Session Management and Validation

Other operations (e.g. sensor, etc.) will be added if necessary.

## Dependency
* Running
    * Oracle VirtualBox 5.x
* Build
    * Go 1.6.2
    
## Build
Please make sure that you have installed Go in your environment, and all the environment variable GOBIN, GOPATH, and GOROOT have been setup properly.

Then:

* Clone this project
```sh
$ cd src
$ git clone https://github.com/rmxymh/infra-ecosphere.git
$ cd infra-ecoshphere
```

* Prepare for dependencies
```sh
$ go get github.com/rmxymh/go-virtualbox
```

* Build
```sh
$ go install
```

Then you can find your executable file is under $GOBIN.

## Execute
### Config File
Config file is written in json format and it describes the following:

* The relationship between BMC IP address and Oracle VirtualBox VM name
* BMC Account

The format is as the following, and if you want more detailed information about it, you can refer to "Configuration" in model/config.go, where the structure that stores the configuration:

```json
{
	"Nodes": [
		{
			"BMCIP": <BMC IP>,
			"VMName": <Virtual Machine Name>
		}
	],
	"BMCUsers": [
		{
			"Username": <BMC Username>
			"Password": <BMC Password>
		}
	]
}
```

More detailed example:

```json
{
	"Nodes": [
		{
			"BMCIP": "127.0.1.1",
			"VMName": "TestVM01"
		},
		{
			"BMCIP": "127.0.1.2",
			"VMName": "TestVM02"
		}
	],
	"BMCUsers": [
		{
			"Username": "admin",
			"Password": "admin"
		}
	]
}
```

It indicate that we have 1 BMC username and password pair, and we have 2 Virtual Machines that we want to map the simulated BMC:

* TestVM01: A Virtual Machine whose simulated BMC IP is 127.0.1.1
* TestVM02: A Virtual Machine whose simulated BMC IP is 127.0.1.2

Here we need to be aware that:

* This project DOESN'T implement any packet forwarding mechanism, and it only implements UDP servers here, so we need to make sure that those IP address can be listened and reachable by your command sender.
* Here we have two suggested methods for assigning BMC IP:
    * IP Alias on interfaces in your machine which runs this program.
    * Use loopback IP address. (127.0.0.0/8)

### Execution

After you have prepared for the configuration file in the same path as your executable file, just execute the executable file:

```sh
$ cd $GOBIN
$ ./infra-ecosphere
```

After the program is running, you can use IPMI related utilities, such as ipmitool, to operate your VM. For instance:

```sh
$ ipmitool -U admin -P admin -H 127.0.1.1 chassis power status
```

## TODO

* Refine the overall project structure.
* Separate IPMI packet management as a library so that every one can use it with callback functions.
* Find a way to make local VM can touch IPMI server in the host machine.
    * Tunnel (Evaluating)
    * Expose BMC operations as a Web Service, and local VM can intercept IPMI packets and convert them into the corresponding API calls. (Evaluating)

