# Openwrt port of Icaro HotSpot

Minimum HW requirements:

| RAM   | Flash |
|-------|-------|
| 32 MB |  8 MB |

Pre compiled architectures:

  * mips_24kc
  * mipsel_24kc

List of compatible devices: https://openwrt.org/supported_devices

Tested OpenWrt releases:

 * OpenWrt 18.06.1
 * LEDE 17.01.5

Tested devices:

* [Asus RT-N14U](https://openwrt.org/toh/asus/rt-n14u)
* [GL-AR300M](https://www.gl-inet.com/products/gl-ar300m/)
* [GL-AR750](https://www.gl-inet.com/products/gl-ar750/)

## Installation

1. Enable https for opkg packages lists.
```shell
	# opkg update
	# opkg install libustream-mbedtls

```
2. Add Icaro repository for your architecture:

To find your architecture use `opkg print-architecture` command, e.g.:
```
root@GL-AR300M:~# opkg print-architecture
arch all 1
arch noarch 1
arch mips_24kc 10
```
The architecture in this case is *mips_24kc*, replace it with every occurrence of the term *ARCH*

```shell
	# echo "src/gz icaro  http://nethesis.github.io/icaro-openwrt_repo/ARCH/icaro" >> /etc/opkg/customfeeds.conf
```
3. Disable signature check commenting the line ``option check_signature 1`` in ``/etc/opkg.conf``

4. Update packages list and install `openwrt-dedalo`:
```shell
	# opkg update
	# opkg install openwrt-dedalo
```

## Network Configuration

Add at least one physical interface to the hotspot network and make sure the is only assigned to hotspot network.

## First setup

**Set up network configuration BEFORE this steps**

1. Generate a UUID using:
 ```shell
	# uuidgen
 ```
 
 
2. Edit `/etc/config/dedalo` with your configurations settings

Configuration example:

```
config dedalo
	option disabled '0'
	option network '192.168.69.0/24'
        option splash_page 'http://icaro.mydomain.com/wings'
        option aaa_url 'https://icaro.mydomain.com/wax/aaa'
        option api_url 'https://icaro.mydomain.com/api'
	option hotspot_id '176'
	option unit_name 'hotelthesea.example.org'
	option unit_description 'MyHotelAtTheSea'
	option unit_uuid '161fre6d-8578-4247-b4a2-c40dced94bdd'
	option secret 'My$uperS3cret'
```

Available options:

- `disabled`: Enable/disable dedalo service (default true)
- `network`: network for clients connected to Dedalo eg: `192.168.69.0/24`
- `splash_page`: Wings (capitve portal) URL hosted on your Icaro installation, eg: ``http://icaro.mydomain.com/wings``
- `aaa_url`:  Wax (Radius over HTTP) URL hosted on your Icaro installation, eg: ``https://icaro.mydomain.com/wax/aaa``
- `api_url`: Sun APIs URL hosted on your Icaro installation, eg: ``https://icaro.mydomain.com/api``
- `hotspot_id`:  the id of the Hotspot already present inside Icaro
- `unit_name`: hostname of local installation, eg: ``hotelthesea.example.org``
- `unit_description`: a descriptive name of local installation, eg: ``MyHotelAtTheSea``
- `unit_uuid`:  a unique unit idenifier, usually a UUID, eg ``161fre6d-8578-4247-b4a2-c40dced94bdd``
- `secret`: a shared secret between this unit and Icaro installation, eg: ``My$uperS3cret``

 
3. Reload dedalo:
 ```shell
	# /etc/init.d/dedalo reload
 ```
4. Register dedalo unit:
 ```shell
	# dedalo register -u <your reseller username> -p <your reseller password>
 ```
5. Restart dedalo:
 ```shell
	# dedalo restart
 ```
6. Some device need a physical restart for change to be apply correctly (some GL-iNet):
 ```shell
	# reboot
 ```

## Development

1. Follow steps on https://openwrt.org/docs/guide-developer/using_the_sdk

2. Add icaro feeds in `feeds.conf.default` file:
 ```
	src-git icaro https://github.com/nethesis/icaro-openwrt.git
 ```
or path to your local clone:
 ```
	src-link icaro /full/path/to/the/local/folder
 ```
3. After `./scripts/feeds update -a` install icaro related packages with:
 ```shell
	$ ./scripts/feeds install openwrt-dedalo
 ```
4. Open configuration menu with `make menuconfig` then select `openwt-dedalo` under Network menu.

5. Compile with `make`

6. The generated repostory with .ipk files are placed in the `bin/packages/<arch>/icaro` directory eg. `bin/packages/mipsel_24kc/icaro`
