I have seen many VirtualBox derivatives<sup>\[1]</sup> checking for Windows hypervisor (the Hyper-V kernel) to avoid incompatibility, in one or more of the following ways:

*	Checking Windows feature installation status via DISM
*	Checking for presence of Hyper-V related userspace processes/services (as seen in StackOverflow)
*	Even worse, `grep`ping pattern `Hyper-V` in output of `systeminfo`

The incompatibility is caused by that the Windows hypervisor does not expose nested virtualization to dom0 and VirtualBox connot provide aan usable VM in WHPX mode until a recent version, so checking for feature/service in dom0 doesn’t help if the flags can’t directly indicate if Windows hypervisor is running.

Unfortunately, all above methods can have false positive and false negative results. Here are some example:

* `bcdedit /set hypervisorlaunchtype off` can stop Hyper-V kernel from booting while keeping all userspace services intact.
*	Enabling one or more Virtualization Based Security services will enable Hyper-V kernel without Windows feature installation (hence no userspace services)

And yes, removing Hyper-V related features may not stop Windows hypervisor from running.

However, starting from Windows 10, Virtualization Based Security is force enabled<sup>\[2]</sup> (maybe with no services configured) if the Windows hypervisor is running. This could be a direct flag to indicate running state of Windows hypervisor.

The status can be queried from WMI, see details in [Microsoft docs].



\[1] Distributed with theirs custom Android-based OS and GUI, you may know what I am saying.

\[2] It is possible to get Windows hypervisor running without Virtualization Based Security, but this needs extra setting on each boot and the setting cannot be made permanent (Thanks to a Microsoft employee who wishes to remain anonymous for confirming it). We could assume the user knows what they are doing.

---

我已经看到不少 VirtualBox 的衍生产品<sup>\[1]</sup>试图通过以下一种或多种方式检查 Windows 虚拟机管理程序（Hyper-V 内核）以避免不兼容。

* 通过 DISM 检查 Windows 功能的安装状态
* 检查 Hyper-V 相关的用户空间进程/服务的存在（正如StackOverflow中看到的）
* 更有甚者，在 `systeminfo` 命令的输出中查找字符串 `Hyper-V`

不兼容的原因是 Hyper-V 内核没有为 dom0 提供嵌套虚拟化功能，而且 VirtualBox 直到近期版本才支持通过 WHPX 运行一个能用的虚拟机，所以如果不能直接反映 Hyper-V 内核是否在运行，在 dom0 中检查功能/服务是不可靠的。

不幸的是，上述方法都可能出现假阳性和假阴性结果，例如：

* `bcdedit /set hypervisorlaunchtype off` 可以阻止 Hyper-V 内核启动，同时所有用户空间服务保持运行。
* 启用一个或多个基于虚拟化的安全性（Virtualization Based Security，VBS）服务将运行 Hyper-V 内核，而无需安装 Hyper-V 功能（因此不会有用户空间服务）

然而，从 Windows 10 开始，如果 Hyper-V 内核正在运行，VBS 将被强制启用<sup>\[2]</sup>（可能是在没有配置安全服务的状态下）。因此我们可以通过查询 VBS 状态。

VBS 状态可以通过 WMI 中查询，详见 [Microsoft docs]。


\[1] 运行定制的 Android 系统和使用自己的 GUI，你可能知道我在说什么。

\[2] 其实可以只运行 Hyper-V 内核而不启用 VBS，但这需要在每次启动时进行额外的设置，而且这个设置不能持久化（感谢一位不愿透露姓名的微软员工的确认）。我们可以假设用户清楚他们正在做什么。

[Microsoft docs]: https://docs.microsoft.com/en-us/windows/security/threat-protection/device-guard/enable-virtualization-based-protection-of-code-integrity#validate-enabled-windows-defender-device-guard-hardware-based-security-features
