---
layout: post
title: runC
categories: [CloudNative]
description: runC
keywords: runC, container
---


# runC
一个命令行工具，用户管理container。

## 1 一些基本操作

runc的所有命令要指定root参数，否则默认都为/run/runc。root参数是用来存储state的。
如对通过containerd建立的container：  
```
runc --root /run/containerd/runc/k8s.io  list
```

runc  state   就是分析root/<containerid>下面的state.json文件
runc  list    就是分析root下所有目录，目录名为containerid，然后也分析state.json文件。
        容器的state：根据state.json中的initprocessstarttime，proc/pid/stat中的starttime，是否存在exec.fifo文件等判断。
               如果不存在proc/pid/stat，或initprocessstarttime与proc/pid/stat中的starttime不相等，或proc/pid/stat中的state为zombie或dead，则为stoped态
               如果存在exec.fifo文件：created状态， 如果不存在则是running态
runc exec：
      cri的exec   先调用shim的exec（只是建process结构），再调用start
           func (c *criService) execInContainer(ctx context.Context, id string, opts execOptions) 

## 2 使用runC的大体流程：
1. shim启动process1 执行runc create
2. runc create启动processA执行runc init，并通过pipe和runc init同步且交互数据 
3. processA启动processB执行clone和unshare等切换namespaces（当前只有pid，network，ipc，uts，mount）。 processA最后要exit
4. processB启动processC让所有namespace最终生效（pid namespace只在儿子才生效）。 processB最后要exit
5. processC再从runc create的process1接收进程配置，根据配置切换root dir，setuid，setgid等动作。然后停下来等待执行user process
6. runc start通知processC执行user process

## 3 执行效果
#### =============进程关系==========================
```
root      2289     1  2 Jul24 ?    /usr/bin/containerd --root /mnt/sda1/var/lib/containerd

root      2815  2289  0 Jul24 ?    containerd-shim 
-namespace k8s.io 
-workdir /mnt/sda1/var/lib/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22 
-address /run/containerd/containerd.sock 
-containerd-binary /usr/bin/containerd

root      2823  2815  0 Jul24 ?    runc 
--root /run/containerd/runc/k8s.io 
--log /run/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22/log.json 
--log-format json 
create 
--bundle /run/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22 
--pid-file /run/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22/init.pid 
--no-pivot 
04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22

root      2832  2823  0 Jul24 ?    runc init
```

#### ==============目录===========================
runc：
    root目录： 主要放state.json文件
    bundle：
             config.json   init.pid   log.json   rootfs


## 4 代码分析

### runc 和libcontainer的交互

#### 1、runc create的实现：
```
factory = libcontainer.New(...)
container = factory.Create(containerid, libcontainer.Config)
libcontainer.Process.Init = true    《--后面的步骤受此影响
container.Start(libcontainer.Process)       《-- 此时会启动一个桩进程，并不执行user process.
```

#### 2、其他runc  xxx的实现
```
factory = libcontainer.New(...)
container = factory.Load(containerid)    《--复用老container
```

##### 2.1、runc start
```
container.Exec()              《--向container发signal，触发执行user process
```
##### 2.2、runc exec         《--在container中执行一个新进程
```
libcontainer.Process.Init = false    《--后面的步骤受此影响。比如执行的runc init进程不会等待，直接执行user process了
container.Run(libcontainer.Process)
```

##### 2.3、runc kill
```
container.Signal(signal, isAll)
```

#### 3、runc run
```
factory = libcontainer.New(...)
container = factory.Create(containerid, libcontainer.Config)
libcontainer.Process.Init = true    《--后面的步骤受此影响
container.Run(libcontainer.Process)     <==等于container.Start + container.Exec（执行了用户进程）。 
                                        与runc exec相同，都执行container.Run，但process里面的init设置相反
```
																  
#### container object

```
// Container is a libcontainer container object.
type Container interface {
	BaseContainer

	// Methods below here are platform specific
	Checkpoint(criuOpts *CriuOpts) error
	Restore(process *Process, criuOpts *CriuOpts) error
	Pause() error
	Resume() error
	// NotifyOOM returns a read-only channel signaling when the container receives an OOM notification.
	NotifyOOM() (<-chan struct{}, error)
	// NotifyMemoryPressure returns a read-only channel signaling when the container reaches a given pressure level
	NotifyMemoryPressure(level PressureLevel) (<-chan struct{}, error)
}
type BaseContainer interface {
    ID() string       // Returns the ID of the container
    Status() (Status, error)     // Returns the current status of the container.
    State() (*State, error)      // State returns the current container's state information.
    Config() configs.Config      // Returns the current config of the container.
    Processes() ([]int, error)   // Returns the PIDs inside this container. The PIDs are in the namespace of the calling process.
    Stats() (*Stats, error)      // Returns statistics for the container.
    Set(config configs.Config) error     // Set resources of container as configured
    Start(process *Process) (err error)  // Start a process inside the container. 此时并不执行user process.
    Exec() error   // Exec signals the container to exec the users process at the end of the init.  向container发signal，触发执行user process
    Run(process *Process) (err error)    // Run immediately starts the process inside the container. 执行了user process
    Destroy() error                      // Destroys the container, 杀掉所有进程
    Signal(s os.Signal, all bool) error  // Signal sends the provided signal code to the container's initial process.
```

### prepareRootfs的实现

prepareRootfs在runc init的StartInitialization中执行，已经是新的mount namespace了

#### 代码框架：

prepareRoot
  如果RootPropagation未设，则设置/为rslave，否则按RootPropagation设置（在pod带双向传播时，rootPropagation会设为share）
  设置离rootfs的最近的mountpoint为private（若rootfs自己就是mountpoint，则为自己）
  对rootfs执行bind操作，转换为mountpoint

mountToRootfs
    mount用户的mount到rootfs目录下各自对应的mountpoint

pivot（切换/）
```
  chdir(rootfs)
    if no_pivot
         Mount(rootfs, "/", "", unix.MS_MOVE, "")     《--把新rootfs移到/上
         unix.Chroot(".")     《---切换/指向rootfs
         Chdir("/")
    else if用户配置了newMountNamespace       <--pivot-root
         unix.Chdir(rootfs)
         unix.PivotRoot(".", ".")
         unix.Fchdir(oldroot)        《--为了替pivot-root补枪的
         //开始umount oldroot
         unix.Mount("", ".", "", unix.MS_SLAVE|unix.MS_REC, "") <--因为有已知问题而不用private
         unix.Unmount(".", unix.MNT_DETACH)
         unix.Chdir("/")
    else
     unix.Chroot(".")
         unix.Chdir("/")
```

##### ====================具体代码：=======================
```
// prepareRootfs sets up the devices, mount points, and filesystems for use
// inside a new mount namespace. It doesn't set anything as ro. You must call
// finalizeRootfs after this function to finish setting up the rootfs.
func prepareRootfs(pipe io.ReadWriter, iConfig *initConfig) (err error) {
	config := iConfig.Config
	if err := prepareRoot(config); err != nil {
		return newSystemErrorWithCause(err, "preparing rootfs")
	}

	for _, m := range config.Mounts {
		for _, precmd := range m.PremountCmds {
			if err := mountCmd(precmd); err != nil {
				return newSystemErrorWithCause(err, "running premount command")
			}
		}

                   把用户要mount的目录mount进来，目的路径都加上rootfs作为前缀（等于基于host的目录结构来mount）
		if err := mountToRootfs(m, config.Rootfs, config.MountLabel); err != nil {     
			return newSystemErrorWithCausef(err, "mounting %q to rootfs %q at %q", m.Source, config.Rootfs, m.Destination)
		}

		for _, postcmd := range m.PostmountCmds {
			if err := mountCmd(postcmd); err != nil {
				return newSystemErrorWithCause(err, "running postmount command")
			}
		}
	}

	setupDev := needsSetupDev(config)

	if setupDev {
		if err := createDevices(config); err != nil {
			return newSystemErrorWithCause(err, "creating device nodes")
		}
		if err := setupPtmx(config); err != nil {
			return newSystemErrorWithCause(err, "setting up ptmx")
		}
		if err := setupDevSymlinks(config.Rootfs); err != nil {
			return newSystemErrorWithCause(err, "setting up /dev symlinks")
		}
	}

	// Signal the parent to run the pre-start hooks.
	// The hooks are run after the mounts are setup, but before we switch to the new
	// root, so that the old root is still available in the hooks for any mount
	// manipulations.
	// Note that iConfig.Cwd is not guaranteed to exist here.
	if err := syncParentHooks(pipe); err != nil {
		return err
	}

	// The reason these operations are done here rather than in finalizeRootfs
	// is because the console-handling code gets quite sticky if we have to set
	// up the console before doing the pivot_root(2). This is because the
	// Console API has to also work with the ExecIn case, which means that the
	// API must be able to deal with being inside as well as outside the
	// container. It's just cleaner to do this here (at the expense of the
	// operation not being perfectly split).

	if err := unix.Chdir(config.Rootfs); err != nil {
		return newSystemErrorWithCausef(err, "changing dir to %q", config.Rootfs)
	}

	if config.NoPivotRoot {
		err = msMoveRoot(config.Rootfs)
	} else if config.Namespaces.Contains(configs.NEWNS) {
		err = pivotRoot(config.Rootfs)              
	} else {
		err = chroot(config.Rootfs)
	}
	if err != nil {
		return newSystemErrorWithCause(err, "jailing process inside rootfs")
	}

	if setupDev {
		if err := reOpenDevNull(); err != nil {
			return newSystemErrorWithCause(err, "reopening /dev/null inside container")
		}
	}

	if cwd := iConfig.Cwd; cwd != "" {
		// Note that spec.Process.Cwd can contain unclean value like  "../../../../foo/bar...".
		// However, we are safe to call MkDirAll directly because we are in the jail here.
		if err := os.MkdirAll(cwd, 0755); err != nil {
			return err
		}
	}

	return nil
}
```

```
func prepareRoot(config *configs.Config) error {
	flag := unix.MS_SLAVE | unix.MS_REC
	if config.RootPropagation != 0 {
		flag = config.RootPropagation     《--该标志，如果runc检查spec中有双向mount传播，此值就会设置为rshare
	}
	if err := unix.Mount("", "/", "", uintptr(flag), ""); err != nil {      《--把当前/目录设为slave，递归
		return err
	}

	《--docker1.12 runc1.0rc2及之前，此处只在no-pivot时才做。见runc #1148
         rootfs的最近一个mountpoint如果是share，则改为private。rootfs本身如果就是mountpoint，则改自己
	// Make parent mount private to make sure following bind mount does
	// not propagate in other namespaces. Also it will help with kernel
	// check pass in pivot_root. (IS_SHARED(new_mnt->mnt_parent))
	if err := rootfsParentMountPrivate(config.Rootfs); err != nil {  
		return err                                                                                         
	}

          把rootfs及其子孙也改为mountpoint
	return unix.Mount(config.Rootfs, config.Rootfs, "bind", unix.MS_BIND|unix.MS_REC, "")  
}

```

```
// pivotRoot will call pivot_root such that rootfs becomes the new root
// filesystem, and everything else is cleaned up.
func pivotRoot(rootfs string) error {
	// While the documentation may claim otherwise, pivot_root(".", ".") is
	// actually valid. What this results in is / being the new root but
	// /proc/self/cwd being the old root. Since we can play around with the cwd
	// with pivot_root this allows us to pivot without creating directories in
	// the rootfs. Shout-outs to the LXC developers for giving us this idea.

	oldroot, err := unix.Open("/", unix.O_DIRECTORY|unix.O_RDONLY, 0)
	defer unix.Close(oldroot)

	newroot, err := unix.Open(rootfs, unix.O_DIRECTORY|unix.O_RDONLY, 0)
	defer unix.Close(newroot)

	// Change to the new root so that the pivot_root actually acts on it.
	if err := unix.Fchdir(newroot); err != nil {
		return err
	}

	if err := unix.PivotRoot(".", "."); err != nil {              <--此时/为new，/proc/self/cwd为old
		return fmt.Errorf("pivot_root %s", err)
	}

	// Currently our "." is oldroot (according to the current kernel code).
	// However, purely for safety, we will fchdir(oldroot) since there isn't
	// really any guarantee from the kernel what /proc/self/cwd will be after a
	// pivot_root(2).
	if err := unix.Fchdir(oldroot); err != nil {
		return err
	}

	// Make oldroot rslave to make sure our unmounts don't propagate to the
	// host (and thus bork the machine). We don't use rprivate because this is
	// known to cause issues due to races where we still have a reference to a
	// mount while a process in the host namespace are trying to operate on
	// something they think has no mounts (devicemapper in particular).
	if err := unix.Mount("", ".", "", unix.MS_SLAVE|unix.MS_REC, ""); err != nil {
		return err
	}
	// Preform the unmount. MNT_DETACH allows us to unmount /proc/self/cwd.
	if err := unix.Unmount(".", unix.MNT_DETACH); err != nil {
		return err
	}

	// Switch back to our shiny new root.
	if err := unix.Chdir("/"); err != nil {
		return fmt.Errorf("chdir / %s", err)
	}
	return nil
}

```


