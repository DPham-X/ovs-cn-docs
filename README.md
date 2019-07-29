Open vSwitch Classifier Node Statistics Gatherer
=============================

### Information
The CN Statistics Gatherer is a module then can be enabled in Open vSwitch in order to obtain packet statistics. 
The packet statistics are captured when packets pass through the Open vSwitch Kernel interface and their data is parsed. These statistics are then held in a list in the kernel where they will be transmitted to the user-space table using the Netlink Protocol. The userspace module of the Classifier Node Statistics Gatherer will periodically dump these statistics to registered OpenFlow controllers.

#### Classifier Node Statistics Gatherer architecture

        +-----+    +------+
        | C1  |    |  C2  |                    Controller
        +---^-+    +--^---+
     +-----------------------------------------------------+
            |         |
         +--+---------+-----+      +------+
         |  CN Userspace    +------>CN    |
         |                  <------+HTable|    Userspace
         +------^--+--------+      +------+
                |  |
     +-----------------------------------------------------+
                |  |
         +------+--v---------+     +-------+
         |  CN Kernel        +-----> CN    |   Kernel
         |                   <-----+ List  |
         +-------------------+     +-------+


#### The current statistics gathered is for the following protocols are: 
##### TCP
	1. Source and Destination IPv4 Addresses
	2. Source and Destination Port Numbers
	3. Protocol
	4. Packet Sizes
##### UDP
	1. Source and Destination IPv4 Addresses
	2. Source and Destination Port Numbers
	3. Protocol
	4. Packet Sizes

### Installation

#### OpenWrt
Installing the Open vSwitch Classifier Node Statistics Gatherer on OpenWRT requires building the kernel image for OpenWRT with the app enabled, compiled and installed.
Currently the latest OpenWRT compatible version of Open vSwitch is 2.7.0.

To install the Open vSwitch Classifier Node Statistics Gather on OpenWRT start by getting the source code of OpenWRT

From the OpenWRT install procedure for Linux
1. Update and install the essential compilation tools
~~~
sudo apt-get update
sudo apt-get install git-core build-essential libssl-dev libncurses5-dev unzip gawk zlib1g-dev
~~~
2. Clone the OpenWRT git tree to some directory
~~~
git clone https://github.com/openwrt/openwrt.git
~~~
3. Download the latest feeds
~~~
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
~~~
4. Make OpenWrt build system
~~~
make menuconfig
~~~
5. Modify the Open vSwitch Makefile
~~~
cd /feeds/packages/net/openvswitch
~~~
Replace this line
~~~
DEPENDS:=+libpcap +libopenssl +librt +kmod-openvswitch @($(SUPPORTED_KERNELS))
~~~
With
~~~
DEPENDS:=+libpcap +libopenssl +librt +kmod-openvswitch +libnl +libgenl @($(SUPPORTED_KERNELS))
~~~
and add this line
~~~
CONFIGURE_ARGS += --enable-cn-stats
~~~
under
~~~
CONFIGURE_ARGS += --enable-shared
~~~
So you will end up with something like this
~~~
define Package/openvswitch-base
$(call Package/openvswitch/Default)
TITLE:=Open vSwitch Userspace Package (base)
DEPENDS:=+libpcap +libopenssl +librt +kmod-openvswitch +libnl +libgenl @($(SUPPORTED_KERNELS))
endef
~~~
and
~~~
CONFIGURE_ARGS += --with-linux=$(LINUX_DIR) --with-rundir=/var/run
CONFIGURE_ARGS += --enable-ndebug
CONFIGURE_ARGS += --disable-ssl
CONFIGURE_ARGS += --enable-shared
CONFIGURE_ARGS += --enable-cn-stats
~~~
6. Copy the ```0100-ovs-cn-stats.patch``` to the ```patches``` folder
7. Install Open vSwitch using
~~~
make menuconfig
~~~
