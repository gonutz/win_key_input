The goal of this document is to gather information about keyboard input on Windows.

[This post](https://blog.molecular-matters.com/2011/09/05/properly-handling-keyboard-input/) recommends using the [Raw Input API](msdn.microsoft.com/en-us/library/ms645543%28v=vs.85%29.aspx) to get [WM_INPUT](https://msdn.microsoft.com/de-de/library/windows/desktop/ms645590(v=vs.85).aspx) messages for key actions. This lets the programmer distinguish between individual keys, e.g. left vs right shift, and also handles special keys like print and num lock.

To use raw input in your application, first enable it with:

```
RAWINPUTDEVICE device;
device.usUsagePage = 0x01;
device.usUsage = 0x06;
device.dwFlags = RIDEV_NOLEGACY;
device.hwndTarget = hWnd;
RegisterRawInputDevices(&device, 1, sizeof(device));
```

[RegisterRawInputDevices](https://msdn.microsoft.com/de-de/library/windows/desktop/ms645600(v=vs.85).aspx) makes the window receive [WM_INPUT](https://msdn.microsoft.com/de-de/library/windows/desktop/ms645590(v=vs.85).aspx) messages.
The device usage page and usages are listed [here](https://docs.microsoft.com/de-de/windows-hardware/drivers/hid/top-level-collections-opened-by-windows-for-system-use).
The [RAWINPUTDEVICE structure](https://msdn.microsoft.com/de-de/library/windows/desktop/ms645565(v=vs.85).aspx) has a dwFlags field where RIDEV_NOLEGACY means that no WM_KEYDOWN, WM_KEYUP or WM_CHAR messages are generated for this window.

Handle the WM_INPUT message in the window proc:

```
case WM_INPUT:
{
	char buffer[sizeof(RAWINPUT)] = {};
	UINT size = sizeof(RAWINPUT);
	GetRawInputData(reinterpret_cast<HRAWINPUT>(lParam), RID_INPUT, buffer, &size, sizeof(RAWINPUTHEADER));

	// extract keyboard raw input data
	RAWINPUT* raw = reinterpret_cast<RAWINPUT*>(buffer);
	if (raw->header.dwType == RIM_TYPEKEYBOARD)
	{
		const RAWKEYBOARD& rawKB = raw->data.keyboard;
		// do something with the data here
	}
}
```