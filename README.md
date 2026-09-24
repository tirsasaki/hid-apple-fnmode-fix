**English** | [Bahasa Indonesia](README.id.md) | [日本語](README.ja.md)

# Fix F-Key Keyboard Mapping on Linux (`hid_apple`)

Some keyboards, such as the Rexus M84X, use Apple-compatible firmware. Linux detects them as Apple keyboards and handles them with the `hid_apple` kernel module, which makes F1–F12 act as media keys by default instead of standard function keys.

This guide sets the `fnmode` parameter of the `hid_apple` module so F1–F12 work as standard function keys by default.

## Contents

- [Symptom](#symptom)
- [Cause](#cause)
- [Check the current state](#check-the-current-state)
- [Try it temporarily](#try-it-temporarily)
- [Make the change permanent](#make-the-change-permanent)
- [Verify the result](#verify-the-result)
- [Troubleshooting](#troubleshooting)
- [Notes](#notes)
- [Contributing](#contributing)

## Symptom

- Pressing F4 (or another F-key) triggers a media or special action instead of producing an F4 input.
- F1–F12 only behave as standard function keys while `Fn` is held down.
- The issue occurs on some non-Apple keyboards that use Apple-compatible firmware.

## Cause

Linux handles these keyboards through the `hid_apple` kernel module. Its `fnmode` parameter controls how the function-key row behaves.

| Value | Behavior |
| --- | --- |
| `0` | `Fn` is disabled; F1–F12 always act as standard function keys and media functions are unavailable |
| `1` | Media keys by default; F-keys only while `Fn` is pressed |
| `2` | F-keys by default; media functions while `Fn` is pressed (the behavior used in this guide) |

## Check the current state

Run the following commands to check whether `hid_apple` is active:

```bash
lsmod | grep hid_apple
dmesg | grep -i apple
```

If `lsmod` prints a `hid_apple` line, continue to the next section. If it prints nothing, your keyboard is not handled by `hid_apple` and this guide does not apply.

## Try it temporarily

To test the behavior immediately without rebooting, reload the module with `fnmode=2`:

```bash
sudo rmmod hid_apple
sudo modprobe hid_apple fnmode=2
```

> [!NOTE]
> This change is lost after a reboot unless you complete the permanent setup below.

> [!TIP]
> Reloading the module may make the keyboard unresponsive for a moment. To avoid this, set the value directly without reloading the module:
>
> ```bash
> echo 2 | sudo tee /sys/module/hid_apple/parameters/fnmode
> ```

Test F4 or another function key before continuing with the permanent setup.

## Make the change permanent

> [!WARNING]
> This configuration applies globally to every keyboard that uses the `hid_apple` module.

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

#### Arch Linux, Manjaro, EndeavourOS, or CachyOS

```bash
sudo mkinitcpio -P
sudo reboot
```

## Verify the result

After rebooting, press F4 and confirm that it produces an F4 input instead of triggering a media or special action.

You can also check the active `fnmode` value:

```bash
cat /sys/module/hid_apple/parameters/fnmode
```

Expected output:

```text
2
```

## Troubleshooting

### `fnmode` is not `2` after reboot

- **Problem:** The verification command does not return `2`.
- **Cause:** The permanent module configuration has not taken effect.
- **Fix:** Confirm that `/etc/modprobe.d/hid_apple.conf` contains `options hid_apple fnmode=2`. Then rebuild initramfs using the command for your distribution and reboot again.

### `fnmode` is `2` but the F-keys still behave unexpectedly

- **Problem:** The value is correct, but the F-row does not behave as expected.
- **Cause:** Some keyboards have a physical Fn Lock switch or an `Fn` key combination that also changes how the F-row works.
- **Fix:** Check your keyboard's manual for an Fn Lock toggle and try switching it.

## Notes

> [!TIP]
> `/etc/modprobe.d/hid_apple.conf` will be lost if the operating system is reinstalled. Consider backing up this file.

## Contributing

Contributions can be proposed through GitHub issues or pull requests.
