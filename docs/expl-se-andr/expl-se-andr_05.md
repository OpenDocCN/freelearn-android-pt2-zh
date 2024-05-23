# 第五章：系统启动

既然我们已经有了适用于 Android 系统的安全增强（SE），我们需要了解如何使用它，并将其置于可用状态。在本章中，我们将：

+   修改日志级别以在调试时获取更多详细信息

+   跟踪与策略加载相关的启动过程

+   调查 SELinux API 和 SELinuxFS

+   更正最大策略版本号的问题

+   应用补丁以加载和验证 NSA 策略

你可能在 第四章，*UDOO 上的安装* 中注意到一些令人不安的错误信息 `dmesg`。为了刷新你的记忆，以下是一些错误信息：

```kt
# dmesg | grep –i selinux
<6>SELinux: Initializing.
<7>SELinux: Starting in permissive mode
<7>SELinux: Registering netfilter hooks
<3>SELinux: policydb version 26 does not match my version range 15-23
...

```

即使启用了 SELinux，看起来我们的系统仍然不是没有错误的。在这一点上，我们需要了解是什么原因导致了这个错误，以及我们可以做些什么来纠正它。在本章结束时，我们应该能够识别 SE for Android 设备在策略加载方面的启动过程，以及如何将策略加载到内核中。然后，我们将解决策略版本错误。

# 策略加载

Android 设备遵循类似于 *NIX 启动序列的启动顺序。引导加载程序启动内核，内核最终执行 init 进程。init 进程负责通过 init 脚本和守护程序中的一些硬编码逻辑来管理设备的启动过程。与所有进程一样，init 在 main 函数有一个入口点。这是第一个用户空间进程开始的地方。通过导航到 `system/core/init/init.c` 可以找到代码。

当 init 进程进入 `main`（参考以下代码摘录）时，它会处理 `cmdline`，挂载一些 `tmpfs` 文件系统，如 `/dev`，以及一些伪文件系统，如 `procfs`。对于 Android 设备的 SE，init 被修改为尽可能在启动过程的早期加载策略到内核中。在 SELinux 系统中，策略不是构建到内核中的；它位于一个单独的文件中。在 Android 中，早期启动时挂载的唯一文件系统是根文件系统，它是构建到 `boot.img` 中的 ramdisk。策略可以在 UDOO 或目标设备上的根文件系统中找到，位于 `/sepolicy`。此时，init 进程调用一个函数从磁盘加载策略并将其发送到内核，如下所示：

```kt
int main(int argc, char *argv[]) {
...
  process_kernel_cmdline();
  unionselinux_callback cb;
  cb.func_log = klog_write;
  selinux_set_callback(SELINUX_CB_LOG, cb);

  cb.func_audit = audit_callback;
  selinux_set_callback(SELINUX_CB_AUDIT, cb);

  INFO("loading selinux policy\n");
  if (selinux_enabled) {
    if (selinux_android_load_policy() < 0) {
      selinux_enabled = 0;
      INFO("SELinux: Disabled due to failed policy load\n");
    } else {
      selinux_init_all_handles();
    }
  } else {
    INFO("SELinux:  Disabled by command line option\n");
  }
…
```

在前面的代码中，你会注意到一个非常友好的日志信息，`SELinux: Disabled due to failed policy load`，并想知道为什么我们在之前运行 `dmesg` 时没有看到这个信息。这段代码在 `init.rc` 中的 `setlevel` 执行之前执行。

默认的 init 日志级别由 `system/core/include/cutils/klog.h` 中 `KLOG_DEFAULT_LEVEL` 的定义设置。如果我们真的想改变它，我们可以修改它，重新构建，并实际看到那条信息。

既然我们已经确定了策略加载的初始路径，那么让我们跟随它通过系统的过程。`selinux_android_load_policy()`函数可以在 Android 版本的`libselinux`中找到，位于 UDOObian 源树中。该库可以在`external/libselinux`中找到，所有 Android 的修改都可以在`src/android.c`中找到。

函数首先挂载一个名为**SELinuxFS**的伪文件系统。如果您回想一下，这是我们在第四章，*在 UDOObian 上的安装*中看到的`/proc/filesystems`中提到的新文件系统之一。在没有挂载`sysfs`的系统上，挂载点是`/selinux`；在挂载了`sysfs`的系统上，挂载点是`/sys/fs/selinux`。

您可以使用以下命令在运行中的系统上检查`mountpoints`：

```kt
# mount | grep selinuxfs 
selinuxfs /sys/fs/selinux selinuxfs rw,relatime 0 0

```

SELinuxFS 是一个重要的文件系统，因为它提供了内核与用户空间之间控制和管理 SELinux 的接口。因此，为了使策略加载工作，必须挂载它。策略加载使用文件系统将策略文件字节发送到内核。这发生在`selinux_android_load_policy()`函数中：

```kt
int selinux_android_load_policy(void)
{
  char *mnt = SELINUXMNT;
  int rc;
  rc = mount(SELINUXFS, mnt, SELINUXFS, 0, NULL);
  if (rc < 0) {
    if (errno == ENODEV) {
      /* SELinux not enabled in kernel */
      return -1;
    }
    if (errno == ENOENT) {
      /* Fall back to legacy mountpoint. */
      mnt = OLDSELINUXMNT;
      rc = mkdir(mnt, 0755);
      if (rc == -1 && errno != EEXIST) {
        selinux_log(SELINUX_ERROR,"SELinux: Could not mkdir:  %s\n",
        strerror(errno));
        return -1;
      }
      rc = mount(SELINUXFS, mnt, SELINUXFS, 0, NULL);
    }
  }
  if (rc < 0) {
    selinux_log(SELINUX_ERROR,"SELinux:  Could not mount selinuxfs:  %s\n",
    strerror(errno));
    return -1;
  }
  set_selinuxmnt(mnt);

  return selinux_android_reload_policy();
}
```

`set_selinuxmnt(car *mnt)`函数改变了`libselinux`中的一个全局变量，以便其他例程可以找到这个重要接口的位置。从那里它调用了另一个辅助函数`selinux_android_reload_policy()`，该函数位于相同的`libselinux` `android.c`文件中。它按优先顺序遍历一个可能的策略位置数组。这个数组定义如下：

```kt
Static const char *const sepolicy_file[] = {
  "/data/security/current/sepolicy",
  "/sepolicy",
  0 };
```

由于此时只挂载了根文件系统，因此它选择了`/sepolicy`。其他路径用于策略的动态运行时重新加载。在获取到策略文件的有效的文件描述符后，系统将其内存映射到它的地址空间，并调用`security_load_policy(map, size)`将其加载到内核中。这个函数定义在`load_policy.c`中。这里，map 参数是指向策略文件开头的指针，size 参数是文件的大小（以字节为单位）：

```kt
int selinux_android_reload_policy(void)
{
  int fd = -1, rc;
  struct stat sb;
  void *map = NULL;
  int i = 0;

  while (fd < 0 && sepolicy_file[i]) {
    fd = open(sepolicy_file[i], O_RDONLY | O_NOFOLLOW);
    i++;
  }
  if (fd < 0) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not open sepolicy:  %s\n",
    strerror(errno));
    return -1;
  }
  if (fstat(fd, &sb) < 0) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not stat %s:  %s\n",
    sepolicy_file[i], strerror(errno));
    close(fd);
    return -1;
  }
  map = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
  if (map == MAP_FAILED) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not map %s:  %s\n",
    sepolicy_file[i], strerror(errno));
    close(fd);
    return -1;
  }

  rc = security_load_policy(map, sb.st_size);
  if (rc < 0) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not load policy:  %s\n",
    strerror(errno));
    munmap(map, sb.st_size);
    close(fd);
    return -1;
  }

  munmap(map, sb.st_size);
  close(fd);
  selinux_log(SELINUX_INFO, "SELinux: Loaded policy from %s\n", sepolicy_file[i]);

  return 0;
}
```

安全加载策略会打开`<selinuxmnt>/load`文件，在我们的案例中是`/sys/fs/selinux/load`。在这个阶段，策略通过这个伪文件写入到内核中：

```kt
int security_load_policy(void *data, size_t len)
{
  char path[PATH_MAX];
  int fd, ret;

  if (!selinux_mnt) {
    errno = ENOENT;
    return -1;
  }

  snprintf(path, sizeof path, "%s/load", selinux_mnt);
  fd = open(path, O_RDWR);
  if (fd < 0)
  return -1;

  ret = write(fd, data, len);
  close(fd);
  if (ret < 0)
  return -1;
  return 0;
}
```

# 修复策略版本

在这一点上，我们对如何将策略加载到内核中有了清晰的认识。这非常重要。SELinux 与 Android 的集成始于 Android 4.0，因此在移植到各种分支和片段时，这会中断，并且经常缺少代码。然而，理解系统的所有部分，无论多么粗略，都将帮助我们在野外遇到问题时进行纠正和发展。这些信息也有助于理解整个系统，因此当需要修改时，您将知道在哪里查找以及事物是如何工作的。在这一点上，我们准备纠正策略版本。

日志和内核配置都很清楚；只支持到 23 的策略版本，我们试图加载 26 的策略版本。这可能是 Android 的一个常见问题，因为内核往往过时。

Google 提供的 4.3 版本 sepolicy 也存在问题。Google 的一些更改使得配置设备变得更加困难，因为他们调整了策略以满足发布目标。实际上，该策略几乎允许所有操作，因此生成的拒绝日志非常少。策略中的一些域通过每个域的宽容声明完全宽容，这些域也有规则允许所有操作，因此不会生成拒绝日志。为了纠正这个问题，我们可以使用来自 NSA 的更完整的策略。将`external/sepolicy`替换为从[`bitbucket.org/seandroid/external-sepolicy/get/seandroid-4.3.tar.bz2`](https://bitbucket.org/seandroid/external-sepolicy/get/seandroid-4.3.tar.bz2)下载的内容。

提取 NSA 的策略后，我们需要更正策略版本。策略位于`external/sepolicy`中，并使用一个名为`check_policy`的工具进行编译。sepolicy 的`Android.mk`文件将不得不将此版本号传递给编译器，因此我们可以在这里进行调整。在文件顶部，我们找到了罪魁祸首：

```kt
...
# Must be <= /selinux/policyvers reported by the Android kernel.
# Must be within the compatibility range reported by checkpolicy -V.
POLICYVERS ?= 26
...

```

由于该变量可以通过`?=`赋值被覆盖，我们可以在`BoardConfig.mk`中重写这个设置。编辑`device/fsl/imx6/BoardConfigCommon.mk`，在文件底部添加以下`POLICYVERS`行：

```kt
...
BOARD_FLASH_BLOCK_SIZE := 4096
TARGET_RECOVERY_UI_LIB := librecovery_ui_imx
# SELinux Settings
POLICYVERS := 23
-include device/google/gapps/gapps_config.mk

```

由于策略在`boot.img`镜像中，因此需要构建策略和`bootimage`：

```kt
$ mmm -B external/sepolicy/
$ make –j4 bootimage 2>&1 | tee logz
!!!!!!!!! WARNING !!!!!!!!! VERIFY BLOCK DEVICE !!!!!!!!!
$ sudo chmod 666 /dev/sdd1
$ dd if=$OUT/boot.img of=/dev/sdd1 bs=8192 conv=fsync

```

弹出 SD 卡，将其插入 UDOО，并启动。

### 提示

前述命令中的第一条应该产生如下日志输出：

```kt
out/host/linux-x86/bin/checkpolicy: writing binary representation (version 23) to out/target/product/udoo/obj/ETC/sepolicy_intermediates/sepolicy

```

在这一点上，通过使用`dmesg`检查 SELinux 日志，我们可以看到以下内容：

```kt
# dmesg | grep –i selinux
<6>init: loading selinux policy
<7>SELinux: 128 avtab hash slots, 490 rules.
<7>SELinux: 128 avtab hash slots, 490 rules.
<7>SELinux: 1 users, 2 roles, 274 types, 0 bools, 1 sens, 1024 cats
<7>SELinux: 84 classes, 490 rules
<7>SELinux: Completing initialization.

```

我们还需要运行的另一个命令是`getenforce`。`getenforce`命令获取 SELinux 的强制状态。它可能处于三种状态之一：

+   **禁用**: 没有加载策略或没有内核支持

+   **宽容**: 加载了策略，设备记录拒绝操作（但不在强制模式）

+   **强制**: 这个状态与宽容状态类似，不同之处在于策略违规会导致 EACCESS 返回给用户空间

在启动 SELinux 系统时，其中一个目标就是达到强制（enforcing）状态。调试时使用宽容（permissive）模式，如下所示：

```kt
# getenforce
Permissive

```

# 概述

在本章中，我们介绍了通过 init 进程加载重要策略的工作流程。我们还更改了策略版本以适应我们的开发努力和内核版本。从那里，我们能够加载 NSA 策略并验证系统已加载它。本章还展示了一些 SELinux API 及其与 SELinuxFS 的交互。在下一章中，我们将检查文件系统，然后继续努力将系统设置为强制模式。
