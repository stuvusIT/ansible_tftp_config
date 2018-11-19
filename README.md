# TFTP config

Generates a tftp pxelinux (vesa)menu.c32 configuration and installs latest syslinux version _(Syslinux includes all stuff required for pxe booting)_.


## Requirements

Any Debian based system should work, tested on Debian 9.
Furthermore a working tftp server is required at the target host.

You can use our [tftp_server](https://github.com/stuvusIT/tftp_server) role to get this job done.

___Some variables like `tftpd_directory` or `tftpd_owner` are shared between both roles.___


## Generated directory structure

* **`{{ tftpd_directory }}`**  _main directory_
	* **`pxelinux.0`**  _This file is loaded at bootup and loads the most accurate configuration_
	* **`ldlinux.c32`**  _Only exists on newer version of syslinux and is required by `pxelinux.0`_
	* **`bin`**  _Directory created by this role and holds all c32 binaries such as menu.c32 or dmitest.c32 and memdisk (this directory is ansible managed and should only contain files copied by this role!)_
	* **`pxelinux.cfg`**  _Directory holds all menu configurations created by this role, old configurations are not deleted automatically, to remove them simply remove the according file there_
	* **`syslinux-*`**  _Contains the unpacked "raw" syslinux files and directories_
	* **`kernels`**  _Directory not created automatically, but it's recommended to create it manually and to store all your kernels for pxe boots there_
	* **`img`**  _Directory not created automatically, but it's recommended to create it manually and to store all your initram filesystems, iso images and so on there_


## Role Variables

For additional information see [syslinux menu.c32](https://www.syslinux.org/wiki/index.php?title=Comboot/menu.c32) and
[Syslinux config](https://www.syslinux.org/wiki/index.php?title=Config).


### Primary

| Option                         | Type                    | Default             | Description                                                                                                         |
|--------------------------------|-------------------------|---------------------|---------------------------------------------------------------------------------------------------------------------|
| `tftpd_directory`              | string                  | `/srv/tftp`         | The path to the directory the pxelinux.cfg config should generate in (this is usual the PXE server root directory)  |
| `tftpd_owner`                  | string                  | `root`              | Owner for the `tftpd_directory` and newly generated files (the user must already exist)                             |
| `tftpd_group`                  | string                  | `tftp`              | Group for the `tftpd_directory` and newly generated files (the group must already exist)                            |
| `tftpd_mode`                   | string                  | `'u=rwX,g=rX,o=rX'` | Set the file/directory permissions to this mode (Please note this option is applied for both files and directories) |
| `tftp_vesamenu`                | boolean                 | `False`             | Use vesamenu.c32 instead of the more simplified menu.c32 binary                                                     |
| `tftp_always_show_boot_prompt` | boolean                 | `True`              | Always show the boot prompt before any c32 menu binary is loaded                                                    |
| `tftp_kbmap`                   | string                  |                     | Specifies the keymap to use at the boot menu                                                                        |
| `tftp_timeout`                 | integer                 | `10`                | Load selected default entry after timeout exceeds                                                                   |
| `tftp_menu`                    | [dict](#tftp_menu_item) | `[]`                | A list of all menu items                                                                                            |

_None of the Options above are required_


### tftp_menu_item

**Key ___string___:** specifies the menu to create, for most scenarios a `default` entry is necessary. For more valid options see the [syslinux pxelinux documentation](https://www.syslinux.org/wiki/index.php?title=PXELINUX#Configuration)

**Values:**

| Option                    | Type                                       | Default                              | Description                                                                    |
|---------------------------|--------------------------------------------|--------------------------------------|--------------------------------------------------------------------------------|
| `vesamenu`                | boolean                                    | `{{ tftp_vesamenu }}`                | Use the vesamenu.c32 instead of menu.c32                                       |
| `always_show_boot_prompt` | boolean                                    | `{{ tftp_always_show_boot_prompt }}` | Always show the boot prompt                                                    |
| `timeout`                 | integer                                    | `{{ tftp_timeout }}`                 | Load selected default entry after timeout exceeds                              |
| `kbmap`                   | string                                     | `{{ tftp_kbmap }}`                   | Specifies the keymap to use at the boot menu                                   |
| `title`                   | string                                     |                                      | Specifies the title for the menu                                               |
| `tabmsg`                  | string                                     |                                      | Specifies the message to display for option editing via &lt;TAB&gt;            |
| `extra_entries`           | list of strings                            |                                      | Additional (vesa)menu.32 entries, see syslinux documentation for valid options |
| `entries`                 | [list of menu entries](#tftp_menu_entries) |                                      | Menu entries to generate                                                       |

_None of the Options above are required_


### tftp_menu_entries

This can be either the string `separator` to add a separator(blank line) between the entry above and below or a dict with the following options:

| Option    | Type                                       | Default/Required              | Description                                                                                                                                                                                          |
|-----------|--------------------------------------------|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `label`   | string                                     | _optional only for `intmenu`_ | Defines the visible label for the boot entry                                                                                                                                                         |
| `intmenu` | [list of menu entries](#tftp_menu_entries) |                               | If set, a submenu with given entries is created, submenus generated by this option are indented by spaces (they are displayed on the same screen)                                                    |
| `submenu` | [list of menu entries](#tftp_menu_entries) |                               | If set, a submenu with given entries is created, this option generates the submenu in a new screen                                                                                                   |
| `help`    | string                                     |                               | If set, this help message is displayed when selected                                                                                                                                                 |
| `tag`     | string                                     |                               | This option is only valid for `submenu` entries and allows to specify a name for the menu (you can use this name to jump to) _this string can only contain characters and the special character_ `_` |
| `kernel`  | string                                     |                               | The kernel to load for this menu entry                                                                                                                                                               |
| `append`  | string                                     |                               | Specifies the kernel commandline                                                                                                                                                                     |

_None of the Options above are required_


## Example Configuration

```yaml
tftp_always_show_boot_prompt: False
tftpd_menu:
  default:
    title: Main menu
    kbmap: de
    extra_entries:
      - "MENU AUTOBOOT Automatic boot selected entry in # second{,s}..."
      - "MENU COLOR screen 1;73;40 #50ffffff #00000000 std"
    entries:
      - label: Live System
        intmenu:
          - label: FreeDos
            kernel: memdisk
            append: iso raw initrd=img/FD12CD.iso
      - separator
      - label: System tools
        intmenu:
          - label: DMI test
            help: Useful to collect hardware information
            kernel: dmitest.c32
      - separator
      - label: System Operations
        submenu:
          - label: reload
            kernel: menu.c32
          - label: Reboot
            kernel: reboot.c32
```


## License

## Author Information

- [Markus Mroch (Mr. Pi)](https://github.com/Mr-Pi) _markus.mroch@stuvus.uni-stuttgart.de_
