# TFTP config

generates a tftp pxelinux vesamenu.c32 configuration


## Requirements


## Role Variables

For additonal information see [syslinux menu.c32](https://www.syslinux.org/wiki/index.php?title=Comboot/menu.c32).

### Primary
| Option                         | Type                              | Default | Description                                                                                                         | Required |
|--------------------------------|-----------------------------------|---------|---------------------------------------------------------------------------------------------------------------------|:--------:|
| `tftp_vesamenu`                | boolean                           | `True`  | Use vesamenu.c32 instead of the more simplified menu.c32 binary                                                     |     N    |
| `tftp_always_show_boot_prompt` | boolean                           | `False` | Always show the boot prompt before any c32 menu binary is loaded                                                    |     N    |
| `tftp_kbmap`                   | string                            |         | Specifies the keymap to use at the boot menu                                                                        |     N    |
| `tftp_menu`                    | [dict](#tftp_menu_item)           | `[]`    | A list of all menu items                                                                                            |     N    |
| `tftp_title`                   | string                            |         | The title of the main menu                                                                                          |     Y    |
| `tftp_hidden`                  | boolean                           | `False` | Do not display the actual menu unless the user presses a key. All that is displayed is a timeout message.           |     N    |
| `tftp_hidden_key`              | [list of dicts](#tftp_hidden_key) |         | List of hidden key bindigs                                                                                          |     N    |
| `tftp_clear`                   | boolean                           | `False` | Clear the screen when exiting the menu, instead of leaving the menu displayed.                                      |     N    |
| `tftp_require_key_press`       | boolean                           | `False` | Exit the menu system immediately unless either the Shift or Alt key is pressed, or Caps Lock or Scroll Lock is set. |     N    |

## Example Playbook

## License

## Author Information

- [Author Name (nickname)](github profile) _your-full-stuvus-email-address@stuvus.uni-stuttgart.de_
