**English** | [Bahasa Indonesia](README.id.md)

# Fix F-Key Keyboard Mapping on Linux (`hid_apple`)

Some third-party keyboards, including the Rexus M84X and other 75% keyboards, use Apple-compatible firmware. On Linux, this can cause F1–F12 to act as multimedia keys instead of standard function keys.

This guide configures the `hid_apple` kernel module so F1–F12 work as standard function keys by default.

<!-- ADDED: Table of contents for easier navigation. -->
## Contents

- [Symptom](#symptom)
- [Cause](#cause)
- [Check the current state](#check-the-current-state)
- [Try it temporarily](#try-it-temporarily)
- [Make the change permanent](#make-the-change-permanent)
- [Verify the result](#verify-the-result)
- [Troubleshooting](#troubleshooting)
- [Tested on](#tested-on)
- [Notes](#notes)
- [License and contributing](#license-and-contributing)

## Symptom

- Pressing F4 opens a browser instead of producing an F4 input.
- F1–F12 trigger media or special actions instead of standard function-key inputs.
- The issue occurs on some non-Apple keyboards that use Apple-compatible firmware.

## Cause

Linux handles these keyboards through the `hid_apple` kernel module. Its `fnmode` parameter controls how the function-key row behaves.

| Value | Behavior |
| --- | --- |
| `0` | Disables F-keys completely; only media functions are available |
| `1` | F-keys are active only while the `Fn` key is pressed |
| `2` | F-keys are active by default—the desired behavior in this guide |

## Check the current state

Run the following commands to check whether `hid_apple` is active:

```bash
lsmod | grep hid_apple
dmesg | grep -i apple
```

If the module is active, continue to the next section.

## Try it temporarily

To test the behavior immediately without rebooting, reload the module with `fnmode=2`:

```bash
sudo rmmod hid_apple
sudo modprobe hid_apple fnmode=2
```

> [!NOTE]
> This change is temporary. After a reboot, the configuration in `/etc/modprobe.d/hid_apple.conf`, once created, will take effect.

Test F4 or another function key before continuing with the permanent setup.

## Make the change permanent

### 1. Create the configuration file

Open `/etc/modprobe.d/hid_apple.conf`:

```bash
sudo nano /etc/modprobe.d/hid_apple.conf
```

Add the following line:

```ini
options hid_apple fnmode=2
```

Save the file and close the editor.

### 2. Rebuild initramfs and reboot

Use the commands for your distribution.

#### Debian, Ubuntu, or Linux Mint

```bash
sudo update-initramfs -u
sudo reboot
```

#### Fedora, RHEL, or CentOS Stream

```bash
sudo dracut --force
sudo reboot
```

#### Arch Linux, Manjaro, or EndeavourOS

```bash
sudo mkinitcpio -P
sudo reboot
```

## Verify the result

After rebooting, press F4 and confirm that it produces an F4 input instead of opening a browser.

You can also check the active `fnmode` value:

```bash
cat /sys/module/hid_apple/parameters/fnmode
```

Expected output:

```text
2
```

<!-- ADDED: Troubleshooting guidance assembled from the existing permanent-configuration and verification steps; no new fix method is introduced. -->
## Troubleshooting

### `fnmode` is not `2` after reboot

- **Problem:** The verification command does not return `2`.
- **Cause:** The permanent module configuration has not taken effect.
- **Fix:** Confirm that `/etc/modprobe.d/hid_apple.conf` contains `options hid_apple fnmode=2`. Then rebuild initramfs using the command for your distribution and reboot again.

## Tested on

| Distribution | Kernel | Keyboard |
| --- | --- | --- |
| Debian 13 | `<fill in>` | `<fill in>` |
| Fedora 44 | `<fill in>` | `<fill in>` |
| Arch Linux | `<fill in>` | `<fill in>` |

## Notes

> [!TIP]
> `/etc/modprobe.d/hid_apple.conf` will be lost if the operating system is reinstalled. Consider backing up this file.

> [!WARNING]
> This configuration applies globally to every keyboard that uses the `hid_apple` module.

<!-- ADDED: The original README does not specify a license or contribution process. -->
## License and contributing

No license is currently specified for this repository. Add a `LICENSE` file to clearly state how others may use, modify, and distribute the project.

Contributions can be proposed through GitHub issues or pull requests.
